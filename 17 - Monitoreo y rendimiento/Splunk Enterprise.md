---
tags:
  - monitoreo
  - splunk
  - siem
  - logs
aliases:
  - Splunk
  - Splunk en Ubuntu
---

# Splunk Enterprise

**Plataforma para ingerir, indexar y analizar datos generados por máquinas (logs, métricas, eventos) en tiempo real. Permite buscar, correlacionar y visualizar la actividad de la infraestructura; muy usada como base de un SIEM.** Esta nota cubre la instalación sobre Ubuntu.

## Requisitos previos

- Una versión soportada de Ubuntu (por ejemplo, 20.04 LTS).
- Espacio en disco y recursos del sistema suficientes.
- Acceso a internet para descargar el paquete.

## Descargar el paquete

1. Crea una cuenta o inicia sesión en `https://www.splunk.com`.
2. En **Free Trials & Downloads** elige **Splunk Enterprise → Get My Free Trial**.
3. Selecciona la versión para **Linux 64 bits** en formato **Debian (.deb)**.
4. Usa la opción **Download via Command Line (wget)** para copiar el comando completo de descarga.

> [!tip] Descarga por terminal
> El botón "Download via Command Line (wget)" entrega un comando listo para pegar en el servidor. Evita tener que descargar el `.deb` en otro equipo y transferirlo.

## Instalación

```bash
# Ir al directorio donde se descargará el paquete
cd Downloads

# Pegar el comando wget copiado desde el sitio de Splunk
# (descarga el .deb)

# Verificar el archivo descargado
ls

# Instalar el paquete
sudo apt install ./splunk<version>.deb
```

> [!info] Versión
> Reemplaza `<version>` por el número de versión real del paquete descargado.

## Primer arranque

```bash
# Inicia aceptando la licencia (solo la primera vez)
sudo /opt/splunk/bin/splunk start --accept-license
```

Acepta la licencia con `y` y, cuando lo solicite, define una **contraseña de administrador** segura.

## Acceso a la interfaz web

```bash
sudo /opt/splunk/bin/splunk start
```

Al terminar de cargar, la terminal muestra la URL ("The Splunk web interface is at ..."), normalmente en el puerto **8000**. Ábrela en el navegador e inicia sesión con el usuario y la contraseña definidos.

> [!warning] Producción
> Cambia la comunidad/credenciales por defecto y restringe el acceso al puerto de la interfaz web mediante firewall. No expongas Splunk directamente a internet sin protección.

Desde el panel ya puedes **ingerir, buscar y analizar** tus datos.

---
📎 Guías: ![[Splunk-Enterprise-Ubuntu.pdf]]

🔗 Relacionado: [[Zabbix y SNMP en MikroTik]] · [[Wireshark]]
