---
tags:
  - programacion
  - python
aliases:
  - SMIC
  - Consola SMIC
  - SMIC LoRa Mesh Console
---

# Scripts SMI (consola y GUI)

**SMIC (Sistema de Monitoreo de Infraestructuras Críticas) es un par de herramientas en Python para comunicarse por puerto serial con nodos LoRa Mesh: una consola de terminal con colores y una interfaz gráfica con tkinter.**

Ambos scripts hablan con un nodo LoRa (placas tipo Heltec V3 o XIAO ESP32-S3) a través de un puerto serie a 115200 baudios, envían comandos al nodo y muestran su salida en tiempo real.

## Dependencias comunes

```bash
pip install pyserial
```

Para la versión GUI también hace falta `tkinter`, que suele venir incluido con Python. En Debian/Ubuntu, si falta:

```bash
sudo apt install python3-tk
```

En Linux, para acceder a los puertos serie sin `sudo`, agrega tu usuario al grupo `dialout` (requiere reiniciar la sesión):

```bash
sudo usermod -a -G dialout $USER
```

## Placas reconocidas (VID/PID)

Ambos scripts detectan automáticamente placas conocidas por su par VID/PID:

| VID/PID            | Chip / placa                  |
| ------------------ | ----------------------------- |
| `0x10C4` / `0xEA60`| Silicon Labs CP2102 (Heltec V3)|
| `0x303A` / `0x1001`| Espressif (XIAO ESP32-S3)     |
| `0x1A86` / `0x7523`| CH340 (ESP32 genérico)        |
| `0x0403` / `0x6001`| FTDI FT232R                   |
| `0x2341` / `0x0043`| Arduino Uno (solo en la GUI)  |

---

## Consola serial (`smic_console.py`)

Consola de terminal profesional con salida coloreada mediante secuencias ANSI. Está pensada para usarse desde una terminal.

### Características

- **Auto-detección de puerto** en Linux (`/dev/ttyUSB*`, `/dev/ttyACM*`) o por VID/PID en Windows/macOS.
- **Selección interactiva** de puerto si hay varias placas.
- **Formateo por colores** según el tipo de línea (errores, beacons, rutas, vecinos, estadísticas, etc.).
- **Estadísticas en tiempo real** del nodo (`NodeStats`): ID, nombre, placa, frecuencia, vecinos, Tx/Rx/Fwd, RSSI, SNR, uptime y heap libre.
- **Registro opcional a archivo** en `~/smic_logs/` con la opción `--log`.

### Uso

```bash
python3 smic_console.py                      # auto-detecta el puerto
python3 smic_console.py --port /dev/ttyUSB0  # puerto explícito (Linux)
python3 smic_console.py --port COM3          # puerto explícito (Windows)
python3 smic_console.py --log                # guarda log en ~/smic_logs/
python3 smic_console.py --list               # lista puertos y sale
```

### Comandos locales de la consola

Los comandos que empiezan con punto se procesan localmente (no se envían al nodo):

| Comando    | Acción                                  |
| ---------- | --------------------------------------- |
| `.help`    | Mostrar ayuda                           |
| `.status`  | Estadísticas en tiempo real del nodo    |
| `.ports`   | Listar puertos disponibles              |
| `.clear`   | Limpiar pantalla                        |
| `.port`    | Mostrar puerto activo                   |
| `.quit`    | Salir de la consola                     |

El resto de comandos (`help`, `info`, `neighbors`, `routes`, `stats`, `beacon`, `broadcast <msg>`, `msg <id> <msg>`, `config ...`) se envían directamente al nodo LoRa.

> [!tip] Un hilo separado (`read_serial`) lee el puerto de forma continua y repinta el prompt, de modo que la salida del nodo no interfiere con lo que estás escribiendo.

---

## Interfaz gráfica (`smic_console_gui.py`)

Aplicación de escritorio basada en **tkinter** con tema oscuro, equivalente gráfico de la consola. Útil cuando se prefiere una ventana con botones a la terminal.

### Características

- **Barra de conexión** con selector de puerto, baudios y botón conectar/desconectar.
- **Reconexión automática** en un hilo dedicado (`SerialReader`) con reintentos cada pocos segundos.
- **Log en tiempo real** coloreado por categoría (verde para mesh/vecinos, cian para recepción, amarillo para estadísticas, rojo para errores).
- **Panel de control** con comando libre, botones de acción rápida (`help`, `info`, `neighbors`, `routes`, `stats`, `beacon`, `screen`, `reboot`), envío de mensajes y configuración LoRa.
- **Configuración LoRa** desde la interfaz: Spreading Factor (SF 7-12), ancho de banda (BW 125/250/500), frecuencia (MHz) y sync word, con botones para aplicar o guardar en flash.
- **Guardado de log a archivo** opcional mediante un diálogo.
- **Barra de estado** con indicador de conexión (punto de color) y contador de paquetes.

### Uso

```bash
python smic_console_gui.py
```

> [!info] La comunicación entre el hilo serial y la interfaz se hace mediante una `queue.Queue`. La GUI la consulta cada 50 ms con `after()`, evitando bloquear el bucle de eventos de tkinter.

### Clasificación de líneas

La función `classify_line` decide el color de cada línea recibida según palabras clave:

```python
def classify_line(line: str) -> str:
    l = line.upper()
    if any(k in l for k in ("BEACON", "VECIN", "NEIGHBOR", "ROUT", "MESH")):
        return "green"
    if any(k in l for k in ("RECV", "MSG:", "RECIBIDO", "RX")):
        return "cyan"
    if any(k in l for k in ("STAT", "INFO", "TX", "UPTIME", "FREE")):
        return "yellow"
    if any(k in l for k in ("ERROR", "FAIL", "ERR:", "WARN", "TIMEOUT")):
        return "red"
    return "gray"
```

---
📎 Archivos: ![[smic_console.py]] · ![[smic_console_gui.py]]

🔗 Relacionado: [[Servidor MQTT a MySQL (Python)]] · [[Ansible]]
