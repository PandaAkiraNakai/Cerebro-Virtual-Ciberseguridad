---
tags:
  - servicios
  - iptv
  - tvheadend
  - streaming
aliases:
  - TVHeadend
  - Servidor IPTV m3u
---

# Servidor IPTV con TVHeadend

**Montaje de un servidor IPTV con TVHeadend sobre Ubuntu para servir canales a partir de una lista m3u, administrándolo por su panel web y consumiéndolo desde un cliente como VLC.**

## 1. Servidor base

Instala un servidor **Ubuntu** (por ejemplo en VirtualBox o en una VM/instancia).

## 2. Instalar TVHeadend

En la terminal de Ubuntu:

```bash
sudo apt update
sudo apt install snapd
sudo snap install tvheadend
```

## 3. Administración web

Accede desde el navegador a la IP del servidor en el puerto **9981**:

```text
http://<ip_servidor>:9981
```

El sistema incluye un **asistente de configuración**: selecciónalo y sigue las indicaciones (allí defines el usuario y la contraseña de acceso).

## 4. Agregar canales m3u

1. Consigue la URL de una lista m3u (existen sitios que recopilan canales gratuitos por país).
2. En **tipos de red** selecciona **"Red automática IPTV"**.
3. Introduce la **URL** de la lista m3u.
4. **Mapea los canales** para dejarlos disponibles.

> [!info] Listas m3u
> Una lista m3u es un archivo de texto que enumera flujos de streaming (URLs). TVHeadend la procesa como una "red IPTV" y genera los canales automáticamente.

> [!warning] Legalidad del contenido
> Usa únicamente listas de canales de emisión libre/gratuita y con derechos de redistribución. Retransmitir contenido protegido sin autorización es ilegal.

## 5. Cliente VLC

En VLC: `Medio → Abrir ubicación de red` e introduce la URL del playlist del servidor, con la IP, el puerto **9981** y la ruta `/playlist`:

```text
http://<ip_servidor>:9981/playlist
```

Introduce el **usuario y contraseña** creados en el asistente de TVHeadend.

> [!tip] Acceso autenticado
> El endpoint `/playlist` pide las credenciales definidas en el asistente. Si VLC no carga los canales, verifica IP, puerto y que el usuario tenga permisos de streaming.

---
📎 Guías: ![[Servidor-IPTV-TVHeadend.pdf]]

🔗 Relacionado: [[PBX (Asterisk - FreePBX) en AWS]] · [[Capacity Planning y QoS]]
