---
tags:
  - cloud
  - aws
  - mysql
  - phpmyadmin
aliases:
  - MySQL en AWS
  - MySQL conexión remota
---

# MySQL + phpMyAdmin

**Instalación de un servidor MySQL en una instancia EC2 de AWS (Ubuntu), habilitación de conexión remota y administración mediante phpMyAdmin. Pensado para almacenar datos de proyectos IoT, pero aplicable a cualquier base de datos relacional.**

## Instalar MySQL

```bash
# Actualizar la máquina
apt-get update

# Instalar el servidor MySQL
apt-get install mysql-server

# Verificar el estado del servicio
systemctl status mysql
```

## Crear base de datos, usuario y tabla

```sql
-- Acceder como root
-- (en CLI: mysql -u root -p)

CREATE DATABASE sensores_db;

-- Usuario accesible desde cualquier host (%)
CREATE USER 'adminsql'@'%' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON sensores_db.* TO 'adminsql'@'%';

USE sensores_db;

CREATE TABLE lecturas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha_hora DATETIME,
    humedad FLOAT,
    temperatura FLOAT
);
```

> [!tip] Diseño de la tabla
> El esquema de la tabla (`lecturas`) es solo un ejemplo para datos de sensores. Ajusta columnas y tipos a las necesidades reales de tu proyecto.

## Habilitar conexión remota

Por defecto MySQL solo escucha en `127.0.0.1`. Para aceptar conexiones externas:

```bash
cd /etc/mysql/mysql.conf.d
nano mysqld.cnf
```

Cambia la línea `bind-address`:

```conf
# Antes
bind-address = 127.0.0.1
# Después (escucha en todas las interfaces)
bind-address = 0.0.0.0
```

Guarda (`CTRL+X` → `Y`) y reinicia:

```bash
systemctl restart mysql
```

> [!warning] Exposición y credenciales
> Abrir `0.0.0.0` y un usuario `'@'%'` deja la base accesible desde cualquier IP. En AWS, restringe el **puerto 3306** en el Security Group a las IPs de confianza y usa contraseñas fuertes. Nunca dejes credenciales por defecto.

## Probar la conexión remota

Desde MySQL Workbench (u otra herramienta) crea una conexión con:

- Nombre de la conexión.
- **IP pública** de la instancia EC2.
- Puerto **3306**.
- Usuario (`adminsql` en el ejemplo) y su contraseña.

## Instalar phpMyAdmin

```bash
sudo apt update
sudo apt upgrade
sudo apt install phpmyadmin
```

Durante la instalación:

- Servidor web: selecciona **apache2** (con la barra `espacio`).
- `dbconfig-common`: selecciona **No**.

Accede desde el navegador en `http://<ip_publica>/phpmyadmin` e inicia sesión con las credenciales de MySQL.

### Opcional: instalar PHP y el módulo de Apache

```bash
sudo apt update
sudo apt install php
sudo apt install libapache2-mod-php
sudo systemctl restart apache2
```

---
📎 Guías: ![[MySQL-phpMyAdmin.pdf]]

🔗 Relacionado: [[AWS - VPC, subredes, balanceo y autoescalado]] · [[Envío de mensajes MQTT desde PHP]] · [[SQL (notas)]]
