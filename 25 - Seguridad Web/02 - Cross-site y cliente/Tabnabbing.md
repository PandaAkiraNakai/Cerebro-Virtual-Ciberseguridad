---
tags:
  - seguridad-web
  - owasp
  - tabnabbing
aliases:
  - Tabnabbing
  - Reverse Tabnabbing
  - Tabnabbing inverso
---

# Tabnabbing

**El reverse tabnabbing es un ataque en el que una página abierta en una pestaña nueva (vía `target="_blank"` o `window.open`) obtiene acceso a `window.opener` y redirige la pestaña original a una página de phishing, aprovechando la confianza del usuario en el sitio que ya tenía abierto.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Cuando un enlace abre otra página en una pestaña nueva sin protección, la página destino recibe una referencia `window.opener` a la pestaña que la abrió. Aunque la *same-origin policy* impide leer su contenido cross-origin, **sí** permite navegarla: `window.opener.location = ...`. El atacante (o un sitio enlazado desde contenido de usuario) cambia la pestaña original por una copia falsa del login; el usuario vuelve, ve la "sesión caducada" y reintroduce sus credenciales.

## Detección y pruebas
Busca enlaces a dominios externos con `target="_blank"` que **no** lleven `rel="noopener"`. Revisa también `window.open(url)` en JavaScript. En navegadores modernos (Chrome 88+, Firefox 79+, Safari) `target="_blank"` implica `noopener` por defecto, pero sigue siendo explotable en navegadores antiguos, en `window.open` explícito y en webviews/apps embebidas.

## Explotación / payloads
**Página maliciosa que redirige al opener:**

```html
<!-- enlazada desde un comentario/perfil de la víctima -->
<a href="https://atacante.com/landing" target="_blank">Mira esto</a>
```

```javascript
// en https://atacante.com/landing
if (window.opener) {
  window.opener.location = "https://banco-falso.com/login";
}
```

**Vía window.open sin opciones de seguridad:**

```javascript
window.open("https://atacante.com");   // hereda opener salvo noopener
```

## Mitigación
- Añadir `rel="noopener noreferrer"` a todo enlace con `target="_blank"` hacia destinos externos.
- En JavaScript: `var w = window.open(url); w.opener = null;` o `window.open(url, '_blank', 'noopener')`.
- No abrir contenido controlado por el usuario en pestañas con acceso al opener.
- Mantener actualizado el navegador (los modernos ya aplican `noopener` implícito en anclas).

## Herramientas
- **Burp Suite** / revisión de código — localizar `target="_blank"` sin `rel`.
- Linters de seguridad front-end (ESLint plugins) que marcan el patrón.

---
🔗 Relacionado: [[Clickjacking]] · [[Open Redirect]] · [[XSS (Cross-Site Scripting)]]
📚 Fuente: [PayloadsAllTheThings — Tabnabbing](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Tabnabbing) (MIT, © Swissky)
