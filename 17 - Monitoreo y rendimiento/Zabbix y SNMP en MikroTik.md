---
tags:
  - monitoreo
  - zabbix
  - snmp
  - mikrotik
aliases:
  - Zabbix Appliance
  - Monitoreo MikroTik con Zabbix
---

# Zabbix y SNMP en MikroTik

**Despliegue de un servidor Zabbix a partir de la imagen oficial (Zabbix Appliance) en VirtualBox y agregado de un router MikroTik como host monitoreado vía SNMP. SNMP está disponible tanto en equipos Cisco como MikroTik, por lo que el flujo es reutilizable.**

## 1. Descargar Zabbix Appliance

Desde el sitio oficial de Zabbix, en el apartado **Zabbix Appliance**, descarga el archivo en formato **Open Virtualization Format (.ovf)**. La propia página incluye el manual con las credenciales por defecto del sistema.

## 2. Importar la imagen en VirtualBox

1. `Archivo → Importar servicio virtualizado` y selecciona el `.ovf` descargado.
2. Finaliza la importación.
3. Configura la red en modo **adaptador puente** y elige la interfaz por la que tu equipo accede a la red local.

> [!info] Por qué adaptador puente
> En modo puente la VM obtiene una IP del mismo segmento que tu red, de modo que pueda alcanzar al router MikroTik y tú puedas alcanzar el panel web de Zabbix.

## 3. Acceso al sistema operativo

Inicia la VM. Las credenciales por defecto del SO (publicadas en el manual oficial) suelen ser:

- usuario: `root`
- contraseña: `zabbix`

Obtén la IP del servidor (debe estar en tu segmento de red local):

```bash
ip a
```

## 4. Administración web

En el navegador, accede a la IP del servidor. Credenciales por defecto del panel:

- usuario: `Admin`
- contraseña: `zabbix`

> [!warning] Cambia las contraseñas por defecto
> Tanto el acceso al SO (`root`/`zabbix`) como el del panel (`Admin`/`zabbix`) son públicos. Cámbialos antes de poner el servidor en uso real.

## 5. Habilitar SNMP en el MikroTik

En Winbox/WebFig: `IP → SNMP`.

1. En **Communities**, cambia el nombre de la comunidad por defecto y aplica.
2. Marca **Enable** para activar SNMP y completa:
   - **Community**: el nombre que definiste.
   - **Version**: la versión de SNMP a utilizar (ej. v1 o v2c).

## 6. Agregar el MikroTik a Zabbix

`Menú → Hosts → Create host`, con estos datos:

- **Host name**: un nombre para el equipo.
- **Templates**: busca el modelo o usa `MikroTik SNMP` (incluye los gráficos preconfigurados).
- **Host groups**: asígnalo a un grupo existente.
- **Interfaces**: tipo **SNMP** + la IP del MikroTik.
- **SNMP version**: la misma configurada en el router.
- **SNMP community**: el nombre de comunidad definido en el MikroTik.

Tras unos minutos, la casilla **SNMP** del host aparecerá en **verde**, indicando que la comunicación se estableció correctamente.

> [!tip] La comunidad debe coincidir
> El nombre de comunidad y la versión deben ser idénticos en el MikroTik y en Zabbix; si no coinciden, la interfaz queda en rojo y no llegan datos.

---
📎 Guías: ![[Zabbix-SNMP-MikroTik.pdf]]

🔗 Relacionado: [[Zabbix y SNMP en Cisco]] · [[Splunk Enterprise]] · [[Wireshark]]
