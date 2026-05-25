---
tags:
  - iot
  - mqtt
  - python
aliases:
  - Shell MQTT
  - Panel MQTT
---

# Panel y comandos MQTT (Python)

**Dos scripts en Python que forman una shell remota sobre MQTT: un intérprete que corre en el equipo objetivo y ejecuta los comandos recibidos, y un panel/cliente que los envía y muestra la respuesta. Útil para administración remota en laboratorio sobre un broker Mosquitto.**

## Arquitectura

```
[ panel-mqtt.py ]  --publish-->  topic "comandos"   --> [ interprete-comandos-mqtt.py ]
[ panel-mqtt.py ]  <--subscribe-- topic "resultados" <-- [ interprete-comandos-mqtt.py ]
```

- **Tópico `comandos`**: el panel publica comandos de shell.
- **Tópico `resultados`**: el intérprete publica la salida del comando.

> [!warning] Esto es efectivamente una **shell remota**. Cualquiera con acceso al broker y a los tópicos puede ejecutar comandos arbitrarios en el equipo que corre el intérprete. Úsalo solo en entornos controlados y con autenticación.

## Intérprete de comandos (lado servidor/objetivo)

Escucha el tópico `comandos`, ejecuta cada mensaje como comando de shell (mantiene el estado de `cd`/`pwd`) y publica la salida en `resultados`.

```python
#!/usr/bin/env python3
import paho.mqtt.client as mqtt
import subprocess
import os

# Configuracion MQTT
BROKER = "0.0.0.0"
PORT = 1883
USER = "TU_USUARIO_MQTT"
PASS = "TU_PASSWORD_MQTT"
TOPIC_COMANDOS = "comandos"
TOPIC_RESULTADOS = "resultados"

# Variable global para mantener el directorio actual
directorio_actual = os.getcwd()

def ejecutar_comando(comando):
    """Ejecuta comandos y mantiene el estado del directorio actual."""
    global directorio_actual
    try:
        # Manejar el comando 'cd'
        if comando.strip().startswith("cd"):
            partes = comando.split(maxsplit=1)
            if len(partes) == 1 or partes[1] == "~":  # 'cd' o 'cd ~'
                directorio_actual = os.path.expanduser("~")
            else:
                nuevo_directorio = os.path.abspath(
                    os.path.join(directorio_actual, partes[1])
                )
                if os.path.isdir(nuevo_directorio):
                    directorio_actual = nuevo_directorio
                else:
                    return f"Error: El directorio '{partes[1]}' no existe"
            return f"Directorio cambiado a: {directorio_actual}"

        if comando.strip() == "pwd":
            return f"Directorio actual: {directorio_actual}"

        # Ejecutar otros comandos en el contexto del directorio actual
        proceso = subprocess.run(
            comando,
            shell=True,
            executable="/bin/bash",
            cwd=directorio_actual,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True,
        )
        return proceso.stdout if proceso.stdout else proceso.stderr
    except Exception as e:
        return f"Error: {str(e)}"

def on_message(client, userdata, msg):
    """Callback para manejar mensajes MQTT."""
    comando = msg.payload.decode()
    resultado = ejecutar_comando(comando)
    client.publish(TOPIC_RESULTADOS, resultado)

# Configuracion del cliente MQTT
client = mqtt.Client()
client.username_pw_set(USER, PASS)
client.on_message = on_message
client.connect(BROKER, PORT)
client.subscribe(TOPIC_COMANDOS)
print("Servidor MQTT listo (soporte completo para comandos 'cd')")
client.loop_forever()
```

## Panel / cliente (lado atacante o administrador)

Shell interactiva con banner ASCII (usa `colorama`). Pide IP, usuario y contraseña del broker, publica los comandos en `comandos` y muestra las respuestas de `resultados`.

```python
import paho.mqtt.client as mqtt
import threading
import time
import sys
import itertools
from colorama import Fore, Style, init
from random import randint

# Inicializar colorama para habilitar colores en la terminal
init(autoreset=True)

def mostrar_banner():
    banner = f"""
{Fore.GREEN}{Style.BRIGHT}
  MQTT  R E M O T E   S H E L L
{Style.RESET_ALL}
{Fore.YELLOW}Conectate, envia comandos y administra...{Style.RESET_ALL}
"""
    print(banner)
    animar_mosquitos()

def animar_mosquitos():
    mosquitos = ['.', '-', 'o', '*', '+', 'x']
    for _ in range(20):
        fila = ''.join([Fore.GREEN + mosquitos[randint(0, len(mosquitos) - 1)] for _ in range(50)])
        print(fila)
        time.sleep(0.02)
    print("\n" * 2)

# Preguntar la IP del broker, usuario y contrasena
BROKER = input(f"{Fore.GREEN}[Configuracion] {Style.RESET_ALL}Ingresa la IP del broker MQTT: ")
PORT = 1883
USERNAME = input(f"{Fore.GREEN}[Autenticacion] {Style.RESET_ALL}Ingresa el usuario del broker: ")
PASSWORD = input(f"{Fore.GREEN}[Autenticacion] {Style.RESET_ALL}Ingresa la contrasena del broker: ")
TOPIC_COMMANDS = "comandos"
TOPIC_RESULTS = "resultados"

connected = False

def on_connect(client, userdata, flags, rc):
    global connected
    if rc == 0:
        print(f"{Fore.LIGHTGREEN_EX}[Conectado] Conectado al broker MQTT")
        connected = True
        client.subscribe(TOPIC_RESULTS)
    else:
        print(f"{Fore.RED}[Error] Error al conectar (codigo {rc})")

def on_message(client, userdata, msg):
    print(f"\n{Style.BRIGHT}[Respuesta]{Style.RESET_ALL} {Fore.YELLOW}{msg.payload.decode('utf-8')}")

def mqtt_loop(client):
    client.loop_start()
    while True:
        if not connected:
            print(f"{Fore.YELLOW}[Reconexion] Intentando reconectar...")
        time.sleep(5)

mostrar_banner()

client = mqtt.Client()
client.username_pw_set(USERNAME, PASSWORD)
client.on_connect = on_connect
client.on_message = on_message

print(f"{Fore.BLUE}[Conectando] {Style.RESET_ALL}Intentando conectar a {BROKER}:{PORT}...")
client.connect(BROKER, PORT)

threading.Thread(target=mqtt_loop, daemon=True, args=(client,)).start()

print(f"{Style.BRIGHT}Bienvenido a la shell MQTT. Escribe '{Fore.CYAN}exit{Style.RESET_ALL}' para salir.")
while True:
    if connected:
        command = input(f"{Fore.LIGHTCYAN_EX}> {Style.RESET_ALL}")
        if command.lower() == "exit":
            print(f"{Fore.BLUE}[Saliendo] Cerrando la conexion...")
            client.loop_stop()
            client.disconnect()
            break
        client.publish(TOPIC_COMMANDS, command)
    else:
        print(f"{Fore.YELLOW}[Esperando] Conexion al broker...")
        time.sleep(1)
```

> [!tip] Dependencias: `pip install paho-mqtt colorama`. El intérprete y el panel deben apuntar al mismo broker y compartir los tópicos `comandos`/`resultados`.

---
🔗 Relacionado: [[MQTT con Mosquitto en Debian]] · [[Raspberry Pi pantalla OLED]]
