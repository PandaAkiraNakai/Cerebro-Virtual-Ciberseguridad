---
tags:
  - meshtastic
  - lora
  - python
  - off-grid
aliases:
  - Meshtastic
  - Meshtastic Control Script
---

# Meshtastic: comandos y scripts

**Meshtastic es una red mesh descentralizada sobre LoRa (Long Range) que envía mensajes y datos entre nodos sin internet ni celular. Esta nota recoge qué es, cómo funciona y scripts en Python para ejecutar comandos remotos sobre la malla y mostrar el estado del nodo en una pantalla OLED.**

## ¿Qué es Meshtastic?

Proyecto de **red mesh descentralizada** sobre **LoRa**. Ideal para emergencias, actividades outdoor o IoT donde no hay cobertura.

### ¿Cómo funciona?
1. **Dispositivos compatibles**: nodos como Heltec V3, TTGO T-Beam, etc., con firmware Meshtastic.
2. **Conexión en malla**: los nodos retransmiten mensajes entre sí, ampliando el alcance.
3. **Interfaz accesible**: se controla por app (Android/iOS) o CLI (Python, Raspberry Pi).
4. **Bajo consumo**: apto para baterías y energía solar.

## Script de control remoto

Un script en Python escucha mensajes de la red Meshtastic que empiecen con `cmd:`, ejecuta el comando en el sistema y responde por la malla con la salida.

- Escucha mensajes `TEXT_MESSAGE_APP` que empiecen con `cmd:`.
- Ejecuta el comando (p. ej. `cmd:uptime`, `cmd:ls`) y responde por la red.
- Registra logs en consola y en `meshtastic_control.log`.

> [!warning] Cualquier nodo de la malla que conozca el formato `cmd:` podrá ejecutar comandos en tu sistema. **Úsalo solo en entornos controlados.** Es, en la práctica, una shell remota sobre LoRa.

### Versión básica (puerto fijo)

```python
#!/usr/bin/env python3
import subprocess
import meshtastic
import meshtastic.serial_interface
from datetime import datetime
import time, logging
from pubsub import pub

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[logging.FileHandler('meshtastic_control.log'), logging.StreamHandler()]
)
logger = logging.getLogger()

def execute_command(command):
    """Ejecuta comandos en el sistema con manejo de errores"""
    try:
        result = subprocess.run(command, shell=True, check=True,
                                stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
        return f"OK {datetime.now().strftime('%H:%M:%S')}:\n{result.stdout[:200]}"
    except subprocess.CalledProcessError as e:
        return f"ERR {datetime.now().strftime('%H:%M:%S')}:\n{e.stderr[:200]}"

def on_receive(packet, interface):
    """Maneja mensajes entrantes manteniendo conectividad inalambrica"""
    try:
        if 'decoded' in packet and packet['decoded']['portnum'] == 'TEXT_MESSAGE_APP':
            message = packet['decoded']['text']
            if message.startswith("cmd:"):
                logger.info(f"Comando recibido: {message}")
                response = execute_command(message[4:])
                interface.sendText(response)
    except Exception as e:
        logger.error(f"Error procesando mensaje: {str(e)}")

def main():
    interface = None
    try:
        interface = meshtastic.serial_interface.SerialInterface(
            devPath="/dev/ttyUSB0",
            noProto=False  # permite operacion paralela de radio y serial
        )
        pub.subscribe(on_receive, "meshtastic.receive")
        logger.info("Sistema listo. Envia comandos con formato: cmd:tu_comando")
        while True:
            time.sleep(1)
    except Exception as e:
        logger.error(f"Error: {str(e)}")
    finally:
        if interface:
            interface.close()

if __name__ == "__main__":
    main()
```

### Versión con autodetección de puerto

Busca automáticamente el primer puerto serial libre (`/dev/ttyACM*` o `/dev/ttyUSB*`) y evita errores si está ocupado.

```python
import glob
import serial

def buscar_puerto_serial():
    """Busca un puerto serial valido y retorna el primero que este libre"""
    posibles_puertos = glob.glob('/dev/ttyACM*') + glob.glob('/dev/ttyUSB*')
    for puerto in posibles_puertos:
        if not puerto_ocupado(puerto):
            return puerto
    return None

def puerto_ocupado(puerto):
    """Devuelve True si el puerto esta ocupado, False si esta libre"""
    try:
        s = serial.Serial(puerto)
        s.close()
        return False
    except serial.SerialException:
        return True

# main() llama a buscar_puerto_serial() y pasa el resultado como devPath
# a SerialInterface; el resto (on_receive, execute_command) es identico
# a la version basica.
```

## Pantalla OLED de estado del nodo

Clase `OLEDMonitor` que muestra en una pantalla SSD1306 el estado de Meshtastic (conectado / nº de nodos), IP de eth0/wlan0, CPU/RAM y la hora. Reintenta la conexión al nodo si se cae.

```python
#!/usr/bin/env python3
from luma.core.interface.serial import i2c
from luma.core.render import canvas
from luma.oled.device import ssd1306
from PIL import ImageFont
import psutil, time, subprocess
import meshtastic.serial_interface
from datetime import datetime

class OLEDMonitor:
    def __init__(self):
        self.serial = i2c(port=1, address=0x3C)
        self.device = ssd1306(self.serial)
        self.font = ImageFont.load_default()
        self.mesh_interface = None
        self.last_error = ""

    def get_interface_ip(self, interface):
        try:
            cmd = f"ip -4 addr show {interface} | grep inet | awk '{{print $2}}' | cut -d/ -f1"
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
            return result.stdout.strip() or "Sin IP"
        except Exception:
            return "Error"

    def connect_meshtastic(self):
        try:
            if self.mesh_interface:
                self.mesh_interface.close()
            self.mesh_interface = meshtastic.serial_interface.SerialInterface(
                devPath="/dev/ttyUSB0", noProto=True)
            self.last_error = ""
            return True
        except Exception as e:
            self.last_error = f"{type(e).__name__}:{str(e)[:15]}"
            return False

    def update_display(self):
        with canvas(self.device) as draw:
            status = "OK" if self.mesh_interface else f"ERR {self.last_error}"
            nodes = len(self.mesh_interface.nodes) if self.mesh_interface else 0
            draw.text((0, 0), f"MESH:{status} NODOS:{nodes}", font=self.font, fill="white")
            draw.text((0, 12), f"ETH:{self.get_interface_ip('eth0')}", font=self.font, fill="white")
            draw.text((0, 24), f"WLAN:{self.get_interface_ip('wlan0')}", font=self.font, fill="white")
            draw.text((0, 36), f"CPU:{psutil.cpu_percent()}% RAM:{psutil.virtual_memory().percent}%",
                      font=self.font, fill="white")
            draw.text((0, 48), datetime.now().strftime("%H:%M:%S"), font=self.font, fill="white")

    def run(self):
        while True:
            if not self.mesh_interface:
                self.connect_meshtastic()
            self.update_display()
            time.sleep(5)

if __name__ == "__main__":
    monitor = OLEDMonitor()
    try:
        monitor.run()
    except KeyboardInterrupt:
        if monitor.mesh_interface:
            monitor.mesh_interface.close()
        monitor.device.clear()
```

## Instalación y uso

```bash
python3 -m venv meshtastic_env
source meshtastic_env/bin/activate
pip install --upgrade pip setuptools wheel
pip install meshtastic pyserial pypubsub
python3 meshtastic_control.py
```

Desde otro nodo, envía un mensaje de texto con el formato:

```
cmd:uptime
```

La salida del comando llega como respuesta por la red Meshtastic.

> [!tip] Si tienes problemas de compilación en Raspberry Pi:
> ```bash
> pip install --no-cache-dir --index-url https://pypi.org/simple meshtastic
> ```

> [!info] Si el puerto queda ocupado: `jobs` para ver procesos suspendidos, libéralo con `kill %n`, o mata el proceso Python con `kill -9 PID`.

---
🔗 Relacionado: [[Raspberry Pi pantalla OLED]] · [[Panel y comandos MQTT (Python)]]
