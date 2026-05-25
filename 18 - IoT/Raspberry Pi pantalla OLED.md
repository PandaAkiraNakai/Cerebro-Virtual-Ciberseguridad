---
tags:
  - iot
  - raspberry
  - python
  - oled
  - mqtt
aliases:
  - OLED Raspberry Pi
  - SSD1306 Raspberry
---

# Raspberry Pi: pantalla OLED

**Cómo mostrar información del sistema (IP, CPU, RAM) en una pantalla OLED I2C SSD1306 conectada a una Raspberry Pi, usando `luma.oled` y `psutil`, y dejarlo corriendo como servicio systemd. Incluye una variante que además muestra el estado de un cliente MQTT.**

## Requisitos

### Hardware
- Raspberry Pi (o cualquier equipo Linux con I2C).
- Pantalla OLED I2C compatible **SSD1306**.
- Cableado I2C correcto: SDA, SCL, GND, VCC (a **3.3 V**, no 5 V).

### Software
- Raspberry Pi OS (o Linux) con Python 3.
- Librerías `luma.oled` (control de la pantalla) y `psutil` (uso de CPU/RAM).

## Paso 1: entorno virtual

```bash
cd ~/proyecto-oled
python3 -m venv oled_env
source oled_env/bin/activate    # veras (oled_env) en el prompt
```

## Paso 2: dependencias

```bash
pip install luma.oled psutil
pip freeze > requirements.txt
```

## Paso 3: script básico (IP + CPU)

Muestra la IP de la interfaz y el uso de CPU, actualizando cada 2 s:

```python
from luma.oled.device import ssd1306
from luma.core.interface.serial import i2c
from luma.core.render import canvas
import time
import psutil
import socket

# Configuracion del dispositivo OLED
serial = i2c(port=1, address=0x3C)
device = ssd1306(serial)

def get_ip_address(interface='eth0'):
    """Obtiene la direccion IP de la interfaz especificada."""
    try:
        interfaces = psutil.net_if_addrs()
        if interface in interfaces:
            for addr in interfaces[interface]:
                if addr.family == socket.AF_INET:  # IPv4
                    return addr.address
        return "Sin IP"
    except Exception:
        return "Error IP"

def get_cpu_usage():
    return psutil.cpu_percent(interval=1)

try:
    while True:
        ip_address = get_ip_address('eth0')  # usa 'wlan0' si vas por Wi-Fi
        cpu_usage = get_cpu_usage()
        with canvas(device) as draw:
            draw.text((0, 0), f"IP: {ip_address}", fill="white")
            draw.text((0, 20), f"CPU: {cpu_usage}%", fill="white")
        time.sleep(2)
except KeyboardInterrupt:
    print("Script detenido manualmente.")
```

## Variante: texto centrado + IP/CPU/RAM

Usa una fuente TrueType (DejaVu) y centra cada línea; alterna con una pantalla de emojis. Muestra IP, CPU y RAM:

```python
from luma.oled.device import ssd1306
from luma.core.interface.serial import i2c
from luma.core.render import canvas
from PIL import ImageFont
import time, psutil, socket

serial = i2c(port=1, address=0x3C)
device = ssd1306(serial)

font_path = "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf"
font = ImageFont.truetype(font_path, 14)

def get_ip_address(interface='eth0'):
    interfaces = psutil.net_if_addrs()
    if interface in interfaces:
        for addr in interfaces[interface]:
            if addr.family == socket.AF_INET:
                return addr.address
    return "Sin IP"

def draw_centered_text(draw, text, y, font, fill="white"):
    w = draw.textlength(text, font=font)
    draw.text(((device.width - w) // 2, y), text, font=font, fill=fill)

while True:
    ip = get_ip_address('eth0')
    cpu = psutil.cpu_percent(interval=None)
    ram = psutil.virtual_memory().percent
    with canvas(device) as draw:
        draw_centered_text(draw, f"IP: {ip}", 0, font)
        draw_centered_text(draw, f"CPU: {cpu}%", 20, font)
        draw_centered_text(draw, f"RAM: {ram}%", 40, font)
    time.sleep(7)
```

## Variante con cliente MQTT

Añade un hilo con cliente MQTT (`paho-mqtt`) que se suscribe al tópico `comandos` y alterna en pantalla entre la info del sistema (CPU/RAM/ETH/WLAN) y el estado MQTT + último comando recibido.

```python
import paho.mqtt.client as mqtt
import threading, time, psutil, socket, logging
from luma.oled.device import ssd1306
from luma.core.interface.serial import i2c
from luma.core.render import canvas
from luma.core.legacy import text
from luma.core.legacy.font import proportional, CP437_FONT

# --- Credenciales MQTT ---
BROKER_MQTT = "IP_DEL_BROKER"
PUERTO = 1883
USUARIO_MQTT = "TU_USUARIO_MQTT"
CONTRASENA_MQTT = "TU_PASSWORD_MQTT"
TOPICO_COMANDOS = "comandos"
# --------------------------

serial = i2c(port=1, address=0x3C)
device = ssd1306(serial, rotate=0)

ultimo_comando = "Esperando comandos"
mqtt_conectado = False

def al_conectar(cliente, userdata, flags, rc):
    global mqtt_conectado
    mqtt_conectado = (rc == 0)
    if rc == 0:
        cliente.subscribe(TOPICO_COMANDOS)

def al_recibir_mensaje(cliente, userdata, msg):
    global ultimo_comando
    if msg.topic == TOPICO_COMANDOS:
        ultimo_comando = f"Cmd: {msg.payload.decode()[:20]}"

def hilo_cliente_mqtt():
    cliente = mqtt.Client()
    cliente.on_connect = al_conectar
    cliente.on_message = al_recibir_mensaje
    cliente.username_pw_set(USUARIO_MQTT, CONTRASENA_MQTT)
    cliente.connect(BROKER_MQTT, PUERTO, 60)
    cliente.loop_forever()

# El bucle principal alterna mostrar_info_sistema() y mostrar_mensajes_mqtt()
# usando luma.core.legacy.text con CP437_FONT. Lanza hilo_cliente_mqtt como daemon.
```

> [!warning] El script MQTT traía broker/usuario/contraseña reales. Sustituidos por `IP_DEL_BROKER`, `TU_USUARIO_MQTT`, `TU_PASSWORD_MQTT`.

## Servicio systemd (arranque automático)

Crea `/etc/systemd/system/run_oled.service`:

```ini
[Unit]
Description=OLED script to display CPU and IP
After=network.target

[Service]
Type=simple
ExecStart=/home/USUARIO/proyecto-oled/oled_env/bin/python /home/USUARIO/proyecto-oled/codigo.py
WorkingDirectory=/home/USUARIO/proyecto-oled
User=USUARIO
Group=USUARIO
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable run_oled.service
sudo systemctl start run_oled.service
sudo systemctl status run_oled.service   # deberia mostrar active (running)
```

## Solución de problemas

| Problema | Solución |
|---|---|
| La pantalla no muestra nada | Verifica el cableado y `sudo i2cdetect -y 1` (debe verse `0x3C` o `0x3D`). Usa 3.3 V, no 5 V. |
| El servicio no arranca | `sudo journalctl -u run_oled.service`; revisa rutas de `ExecStart`/`WorkingDirectory`. |
| IP incorrecta | Cambia la interfaz `eth0` por `wlan0` en `get_ip_address()`. |
| La pantalla parpadea | Aumenta el `time.sleep()` del bucle (p. ej. a 5 s). |

> [!tip] `i2cdetect -y 1` es tu mejor amigo para confirmar que la pantalla está en el bus I2C antes de depurar el código.

---
🔗 Relacionado: [[Panel y comandos MQTT (Python)]] · [[MQTT con Mosquitto en Debian]] · [[Meshtastic comandos y scripts]]
