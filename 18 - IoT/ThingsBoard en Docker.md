---
tags:
  - iot
  - thingsboard
  - docker
aliases:
  - ThingsBoard Docker
---

# ThingsBoard en Docker

**Guía resumida para instalar ThingsBoard (plataforma IoT de visualización y gestión de dispositivos) mediante Docker en Raspberry Pi o Ubuntu. ThingsBoard recibe la telemetría de los ESP8266 por MQTT/HTTP y la muestra en dashboards.**

## Requisitos

- SO: Raspberry Pi OS 64-bit (Bookworm) o Ubuntu 20.04+.
- Docker y Docker Compose instalados.
- Acceso root o usuario con `sudo`.

## Instalar Docker (si falta)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo usermod -aG docker $USER
newgrp docker
```

## Clonar el repositorio oficial

```bash
git clone https://github.com/thingsboard/thingsboard.git
cd thingsboard/docker
```

## Archivo `.env` (opcional)

Puertos por defecto:

- HTTP: 8080 → 9090
- MQTT: 1883
- COAP: 5683

Ejemplo de `.env` (guardar en `thingsboard/docker/.env`):

```env
# Puertos
HTTP_PORT=8080
MQTT_PORT=1883
COAP_PORT=5683
LWM2M_PORT=5685
SNMP_PORT=162

# Usuario administrador de ThingsBoard
TB_ADMIN_USER=admin@thingsboard.org
TB_ADMIN_PASSWORD=admin

# Configuracion de demo
DEMO=true
DEMO_TOKEN=TU_TOKEN_DEMO

# Base de datos PostgreSQL
POSTGRES_DB=thingsboard
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_HOST=tb-postgres
POSTGRES_PORT=5432

# Volumenes de datos
DATA_DIR=./data
LOGS_DIR=./logs
```

## Iniciar ThingsBoard

```bash
docker compose up -d
# o, con la version antigua:
docker-compose up -d
```

Accede vía navegador a `http://TU_IP_LOCAL:8080` con el usuario por defecto:

- **admin@thingsboard.org**
- **admin**

> [!warning] Cambia las credenciales por defecto (`admin/admin`) y los tokens demo antes de exponer ThingsBoard fuera del laboratorio.

## Operación

```bash
docker ps                 # estado de contenedores
docker logs -f thingsboard # ver logs en vivo
docker compose down        # detiene y elimina contenedores
docker compose restart     # reinicia
docker compose down -v     # borra contenedores y volumenes (desinstalar)
```

## Problemas comunes

| Error | Solución |
|---|---|
| `ERR_CONNECTION_REFUSED` | Verifica que el contenedor corre: `docker ps`. |
| `ModuleNotFoundError` en Python | Usa entorno virtual o `pip install --break-system-packages`. |
| No carga en el navegador | Espera 2-3 min tras `docker compose up`, o revisa los logs. |

> [!info] Probado en Raspberry Pi 3 / 4 (aarch64) y Ubuntu 22.04 LTS.

---
🔗 Relacionado: [[ESP8266 sketches y firmware]] · [[MQTT con Mosquitto en Debian]]
