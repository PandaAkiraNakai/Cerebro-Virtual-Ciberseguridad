---
tags:
  - iot
  - mqtt
  - mosquitto
  - debian
aliases:
  - Mosquitto MQTT
  - Broker MQTT
---

# MQTT con Mosquitto en Debian

**Mosquitto es un broker MQTT de código abierto (proyecto Eclipse), ligero y muy usado en IoT y mensajería en tiempo real. Esta nota cubre qué es MQTT, la instalación del broker en Debian 12 y pruebas básicas de publicación/suscripción.**

## ¿Qué es un broker MQTT?

Un broker MQTT es un servidor que gestiona la comunicación entre dispositivos usando el protocolo **MQTT** (Message Queuing Telemetry Transport). Recibe mensajes de los publicadores (*publishers*) y los distribuye a los suscriptores (*subscribers*) según los tópicos (*topics*) configurados.

### Características de Mosquitto
- **Ligero**: ideal para dispositivos con recursos limitados.
- Soporta **MQTT 3.1, 3.1.1 y 5.0**.
- **Open Source** con comunidad activa.
- **Seguro**: autenticación (usuario/contraseña) y cifrado (TLS/SSL).
- **Multiplataforma**: Linux, Windows, macOS y Raspberry Pi.

### Casos de uso
- IoT: sensores y dispositivos inteligentes.
- Domótica (iluminación, climatización).
- Telemetría: monitoreo remoto.
- Chats simples en tiempo real.

## Instalación en Debian 12

```bash
# Actualizar repositorios
sudo apt update

# Servidor Mosquitto
sudo apt install mosquitto

# Cliente Mosquitto (mosquitto_pub / mosquitto_sub)
sudo apt install mosquitto-clients
```

## Configuración

Edita el archivo de configuración:

```bash
sudo nano /etc/mosquitto/mosquitto.conf
```

Agrega al final estas dos líneas para habilitar el listener y el acceso anónimo:

```conf
listener 1883 0.0.0.0
allow_anonymous true
```

Ejemplo del archivo completo:

```conf
# Place your local configuration in /etc/mosquitto/conf.d/
pid_file /run/mosquitto/mosquitto.pid

persistence true
persistence_location /var/lib/mosquitto/

log_dest file /var/log/mosquitto/mosquitto.log

include_dir /etc/mosquitto/conf.d

listener 1883 0.0.0.0
allow_anonymous true
```

Reinicia el servicio para aplicar los cambios:

```bash
sudo systemctl restart mosquitto
```

> [!warning] `allow_anonymous true` deja el broker abierto a cualquiera en la red. Úsalo solo en laboratorio; en producción configura usuarios/contraseñas (`mosquitto_passwd`) y, si es posible, TLS.

## Pruebas: servidor y cliente

Abre dos terminales. En la primera (suscriptor) escucha un tópico:

```bash
mosquitto_sub -h IP_DEL_BROKER -t "test" -v
```

- `-h`: dirección IP del broker MQTT.
- `-t`: tópico al que te suscribes.
- `-v`: modo *verbose* (muestra tópico + mensaje).

En la segunda (publicador) envía un mensaje al mismo tópico:

```bash
mosquitto_pub -h IP_DEL_BROKER -t test -m "hola"
```

- `-m`: mensaje a enviar.

Deberías ver `hola` aparecer en la terminal del suscriptor.

> [!tip] Para procesar la telemetría desde Python, usa la librería **Paho MQTT** (`pip install paho-mqtt`). El flujo típico IoT es: ESP8266/Arduino → broker Mosquitto → script Python → base de datos (p. ej. MySQL en una instancia AWS).

## Integración con base de datos (MySQL + phpMyAdmin)

La guía de MySQL/phpMyAdmin describe el almacenamiento de la telemetría en una instancia AWS EC2:

- Instalar `mysql-server` y crear la base `sensores_db`.
- Crear un usuario remoto y otorgarle privilegios:

```sql
CREATE DATABASE sensores_db;
CREATE USER 'TU_USUARIO'@'%' IDENTIFIED BY 'TU_PASSWORD';
GRANT ALL PRIVILEGES ON sensores_db.* TO 'TU_USUARIO'@'%';
USE sensores_db;

CREATE TABLE lecturas (
  id INT AUTO_INCREMENT PRIMARY KEY,
  fecha_hora DATETIME,
  humedad FLOAT,
  temperatura FLOAT
);
```

- Permitir conexiones remotas en `/etc/mysql/mysql.conf.d/mysqld.cnf` cambiando `bind-address` de `127.0.0.1` a `0.0.0.0`, luego `systemctl restart mysql`.
- Instalar phpMyAdmin (`sudo apt install phpmyadmin`, servidor Apache2) y acceder vía `http://TU_IP/phpmyadmin`.

> [!info] El detalle completo de la integración IoT → MQTT → Python → MySQL/AWS está en la guía adjunta.

---
📎 Guías: ![[Guia-Mosquitto-Debian12.pdf]] · ![[Guia-MySQL-phpMyAdmin.pdf]]

🔗 Relacionado: [[Panel y comandos MQTT (Python)]] · [[ThingsBoard en Docker]] · [[ESP8266 sketches y firmware]]
