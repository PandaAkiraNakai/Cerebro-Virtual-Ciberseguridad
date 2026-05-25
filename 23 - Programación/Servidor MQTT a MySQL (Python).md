---
tags:
  - programacion
  - python
  - mqtt
aliases:
  - MQTT a MySQL
  - Servidor MQTT MySQL
  - mqtt_to_mysql
---

# Servidor MQTT a MySQL (Python)

**Script en Python que se suscribe a un tópico de un broker MQTT (Mosquitto) y almacena cada mensaje recibido en una tabla MySQL, ideal para registrar datos de sensores IoT.**

## Propósito

El script actúa como puente entre una red de sensores que publican vía MQTT y una base de datos relacional:

1. Verifica que el broker **Mosquitto** esté corriendo (a modo informativo).
2. Crea la tabla `mqtt_messages` si no existe.
3. Se conecta al broker MQTT y se suscribe a un tópico.
4. Por cada mensaje recibido, intenta parsearlo como JSON y lo inserta en MySQL con su `timestamp`, tópico, QoS y flag de retención.

## Dependencias

```bash
# Crear entorno virtual
python3 -m venv mqtt_env
source mqtt_env/bin/activate

# Paquetes del sistema (broker MQTT y herramientas)
sudo apt-get update
sudo apt-get install -y mosquitto mosquitto-clients python3-pip

# Librerías de Python
pip install paho-mqtt mysql-connector-python
```

Iniciar y habilitar el broker Mosquitto:

```bash
sudo systemctl start mosquitto
sudo systemctl enable mosquitto
```

## Uso

```bash
python3 mqtt_to_mysql.py
```

El servicio se mantiene escuchando hasta que se interrumpe con `Ctrl+C`. Toda la actividad se registra tanto en consola como en el archivo `mqtt_mysql.log`.

> [!warning] Define la configuración de MySQL y MQTT con placeholders y nunca subas credenciales reales al control de versiones. Para producción usa variables de entorno o un archivo de configuración fuera del repositorio.

## Configuración

Ajusta los diccionarios `MYSQL_CONFIG` y `MQTT_CONFIG` antes de ejecutar:

```python
# Configuración MySQL
MYSQL_CONFIG = {
    'host': 'TU_HOST',
    'user': 'TU_USUARIO',
    'password': 'TU_PASSWORD',
    'database': 'sensores_db',
    'port': 3306
}

# Configuración MQTT
MQTT_CONFIG = {
    'host': 'localhost',   # Broker local
    'port': 1883,
    'keepalive': 60,
    'topic': 'tu_topico',  # Tópico específico
    'qos': 0
}
```

## Estructura de la tabla

El script crea automáticamente la tabla destino con índices sobre `topic` y `timestamp`:

```python
def create_table():
    """Crea la tabla si no existe"""
    conn = mysql.connector.connect(**MYSQL_CONFIG)
    cursor = conn.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS mqtt_messages (
        id INT AUTO_INCREMENT PRIMARY KEY,
        timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
        topic VARCHAR(255) NOT NULL,
        message TEXT NOT NULL,
        qos TINYINT NOT NULL,
        retain BOOLEAN NOT NULL DEFAULT FALSE,
        INDEX(topic),
        INDEX(timestamp)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
    """)
    conn.commit()
```

## Lógica de recepción

El callback `on_message` se ejecuta por cada mensaje. Intenta interpretar el payload como JSON; si no lo es, lo guarda como texto plano:

```python
def on_message(client, userdata, msg):
    conn = mysql.connector.connect(**MYSQL_CONFIG)
    cursor = conn.cursor()

    # Intentar parsear JSON
    try:
        payload = json.loads(msg.payload.decode())
        message = json.dumps(payload, ensure_ascii=False)
    except Exception:
        message = msg.payload.decode()

    cursor.execute("""
    INSERT INTO mqtt_messages
    (timestamp, topic, message, qos, retain)
    VALUES (%s, %s, %s, %s, %s)
    """, (
        datetime.now(),
        msg.topic,
        message,
        msg.qos,
        msg.retain
    ))
    conn.commit()
```

## Bucle principal

`main()` verifica el broker, asegura la tabla, configura los callbacks y arranca el loop MQTT en segundo plano:

```python
def main():
    verify_mosquitto()   # Verificación informativa de Mosquitto
    create_table()       # Crear/verificar tabla MySQL

    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message

    client.connect(MQTT_CONFIG['host'], MQTT_CONFIG['port'], MQTT_CONFIG['keepalive'])
    client.loop_start()

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        client.loop_stop()
        client.disconnect()
```

> [!tip] El conector se abre y cierra en cada mensaje. Para cargas altas conviene mantener un pool de conexiones o reutilizar la conexión, ya que abrir una conexión MySQL por mensaje es costoso.

---
🔗 Relacionado: [[Scripts SMI (consola y GUI)]] · [[Ansible]]
