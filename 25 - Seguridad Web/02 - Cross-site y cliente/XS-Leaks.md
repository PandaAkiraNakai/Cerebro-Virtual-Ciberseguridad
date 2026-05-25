---
tags:
  - seguridad-web
  - owasp
  - xs-leaks
aliases:
  - XS-Leaks
  - XS-Leak
  - Cross-Site Leaks
  - Fugas cross-site
---

# XS-Leaks

**Los XS-Leaks (Cross-Site Leaks) son una familia de ataques de canal lateral que infieren información sobre el estado de un usuario en otro sitio observando efectos secundarios de las respuestas (tiempos, errores, conteos, tamaños), sin llegar a leer el contenido cross-origin directamente.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
La same-origin policy impide leer respuestas cross-origin, pero deja observables muchos **efectos colaterales**: si una imagen carga o falla, cuánto tarda una petición, cuántos frames tiene una página, si una búsqueda devolvió resultados. Una página atacante hace que el navegador de la víctima realice peticiones a un sitio donde está autenticada y mide esas señales binarias ("sí/no") para deducir datos privados: si el usuario está logueado, si existe un recurso, o el resultado de una búsqueda (oráculo de búsqueda).

## Detección y pruebas
Identifica respuestas cuyo comportamiento dependa del estado del usuario: códigos de estado distintos, redirecciones, longitudes variables, número de frames o disponibilidad de un recurso. Cada diferencia observable cross-origin es un canal potencial. Las técnicas clásicas: eventos `onload`/`onerror`, `frame count` (`window.frames.length`), tamaño de caché, y APIs como `performance` o `getComputedStyle`.

## Explotación / payloads
**Oráculo por error de estado** (existe/no existe → carga/falla):

```html
<img src="https://victima.com/api/user/123/avatar"
     onload="existe()" onerror="noExiste()">
```

**Conteo de frames** (revela estado de búsqueda/login):

```javascript
const w = window.open("https://victima.com/buscar?q=secreto");
setTimeout(() => leak(w.frames.length), 1500);  // nº de iframes = resultados
```

**Timing attack** (respuesta autenticada tarda distinto):

```javascript
const t0 = performance.now();
await fetch("https://victima.com/perfil", {mode:"no-cors", credentials:"include"});
const dt = performance.now() - t0;   // umbral -> estado
```

**Otras señales:** sondeo de caché, `Cache-Control` por usuario, errores de CSP que delatan redirecciones, `MessageEvent` de postMessage mal aislado.

## Mitigación
- **Fetch Metadata** (`Sec-Fetch-Site`/`Sec-Fetch-Mode`): rechazar peticiones cross-site no esperadas.
- **SameSite=Lax/Strict** en cookies para cortar el envío en contexto cross-site.
- **Cross-Origin-Opener-Policy** (`same-origin`) y **Cross-Origin-Resource-Policy** para aislar la ventana y los recursos.
- **CORP/COEP + cross-origin isolation**; respuestas de tamaño/tiempo constantes en endpoints sensibles.

## Herramientas
- **XSinator** — banco de pruebas y catálogo de XS-Leaks conocidos.
- **Burp Suite** — medición de tiempos y diferencias de respuesta.

---
🔗 Relacionado: [[Inyección CSS]] · [[Clickjacking]] · [[CORS mal configurado]] · [[DOM Clobbering]]
📚 Fuente: [PayloadsAllTheThings — XS-Leak](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XS-Leak) (MIT, © Swissky)
