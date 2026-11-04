---
tags:
  - monitoreo
  - zabbix
  - mysql
  - linux
aliases:
  - Instalar Zabbix
  - Zabbix 7.4
  - Zabbix Server
---

# Zabbix 7.4 con MySQL

Instalación de **Zabbix Server 7.4** con MySQL 8.0 y Apache 2.4 en **Ubuntu 24.04 LTS**. Requisitos mínimos: 2 GB de RAM y 10 GB de disco libres.

## 1. MySQL y base de datos

```bash
sudo apt update
sudo apt install -y mysql-server
sudo systemctl enable --now mysql
sudo mysql
```

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'ClaveZabbixDB';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;   -- solo durante la importación
QUIT;
```

## 2. Repositorio y paquetes de Zabbix

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_latest_7.4+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

## 3. Esquema inicial

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | \
  mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
sudo mysql -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

## 4. Contraseña de la base en el servidor

En `/etc/zabbix/zabbix_server.conf`:

```ini
DBPassword=ClaveZabbixDB
```

## 5. Arranque

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

Abre `http://IP_SERVIDOR/zabbix`, completa el asistente y entra con `Admin` / `zabbix`.

## Verificación

```bash
sudo systemctl status zabbix-server zabbix-agent apache2
mysql -uzabbix -p -e "USE zabbix; SHOW TABLES;" | head
tail -f /var/log/zabbix/zabbix_server.log
```

## Problemas frecuentes

| Error | Solución |
|---|---|
| `Can't connect to local MySQL server through socket` | `sudo systemctl start mysql` |
| `Access denied for user 'zabbix'` | La clave de `DBPassword` no coincide con la de MySQL |
| La web no responde | `sudo systemctl restart apache2` y revisa el firewall |
| `Zabbix server is not running` en el panel | Revisa `zabbix_server.log`; casi siempre es la conexión a la base |

## Endurecimiento posterior

1. Cambia la contraseña de `Admin` en la interfaz web.
2. Deja MySQL solo en local: `bind-address = 127.0.0.1` en `/etc/mysql/mysql.conf.d/mysqld.cnf`.
3. Abre solo lo necesario: `sudo ufw allow 80/tcp`, y `10051/tcp` si hay agentes activos o proxies.

## En Docker

La imagen `zabbix/zabbix-server-mysql` contiene **solo el servidor**: no trae MySQL ni la interfaz web. Un despliegue en contenedores necesita al menos tres piezas.

| Contenedor | Imagen | Puerto |
|---|---|---|
| Base de datos | `mysql:8.0` | 3306 (interno) |
| Servidor | `zabbix/zabbix-server-mysql` | 10051 |
| Interfaz web | `zabbix/zabbix-web-nginx-mysql` | 8080 |

El repositorio oficial `zabbix/zabbix-docker` trae archivos Compose listos. Para un nodo único dentro de [[GNS3 - laboratorios de red virtualizados|GNS3]], arma una imagen propia que levante las tres piezas, o conecta el nodo de GNS3 a un Zabbix que corra fuera de la topología.

---
🔗 Relacionado: [[SNMPv3 en Cisco]] · [[Zabbix y SNMP en Cisco]] · [[Zabbix y SNMP en MikroTik]] · [[MySQL + phpMyAdmin]] · [[Comandos Docker]] · [[06 - Servicios con systemd]]
