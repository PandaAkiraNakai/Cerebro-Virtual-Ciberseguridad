---
tags:
  - seguridad-web
  - owasp
  - web-cache-deception
aliases:
  - Web Cache Deception
  - WCD
  - Engaño de caché web
---

# Web Cache Deception

**El Web Cache Deception engaña a una caché (CDN, proxy) para que almacene una respuesta con datos privados de un usuario haciéndola pasar por un recurso estático cacheable, de modo que otro usuario (el atacante) recupere luego ese contenido sensible.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Las cachés deciden qué guardar según reglas heurísticas: extensión del archivo (`.css`, `.js`, `.jpg`), prefijo de ruta o cabeceras. El atacante construye una URL que el **origen** interpreta como la página dinámica del usuario (por path normalization), pero que la **caché** ve como estático. Si convence a la víctima de visitar `cuenta/info.css`, el servidor sirve su perfil privado y la caché lo guarda bajo esa URL; luego el atacante pide la misma URL y obtiene los datos de la víctima. La causa raíz es la **discrepancia de parseo** entre caché y origen.

## Detección y pruebas
Añade un sufijo estático falso a una página autenticada (`/perfil` → `/perfil/x.css`, `/perfil%0A.css`, `/perfil;x.js`) y comprueba: (1) que el origen siga devolviendo el contenido dinámico, y (2) cabeceras de caché (`X-Cache: HIT`, `Age`, `CF-Cache-Status`). Si una segunda petición sin sesión devuelve los datos del usuario, hay WCD. Prueba delimitadores (`;`, `%2e`, `%2f`, `%00`, salto de línea) que el origen ignore pero la caché no.

## Explotación / payloads
**Sufijos estáticos y delimitadores** (path confusion):

```
https://victima.com/cuenta/perfil.css
https://victima.com/cuenta/perfil/nonexistent.js
https://victima.com/cuenta/perfil%0Anonexistent.css
https://victima.com/cuenta/perfil;nonexistent.css
https://victima.com/cuenta/perfil%2fnonexistent.css
```

**Flujo del ataque:**

```
1. Atacante envía a la víctima: https://victima.com/cuenta/perfil.css
2. Víctima (logueada) la abre -> origen sirve su perfil, caché lo guarda
3. Atacante pide la MISMA URL (sin sesión) -> recibe el perfil cacheado
```

## Mitigación
- Configurar la caché para guardar **solo por `Content-Type` real**, no por extensión de la URL.
- Homogeneizar la normalización de rutas entre CDN/proxy y origen (eliminar la discrepancia de parseo).
- Marcar respuestas con datos de usuario como `Cache-Control: no-store, private`.
- Restringir el cacheo a una lista explícita de rutas/recursos verdaderamente estáticos.

## Herramientas
- **Burp Suite** (+ extensión *Web Cache Deception*) — pruebas de sufijos y verificación de HIT.
- **Param Miner** — descubrimiento de comportamientos de caché y cache poisoning.

---
🔗 Relacionado: [[Open Redirect]] · [[HTTP Request Smuggling]] · [[Reverse Proxy mal configurado]] · [[SSRF (Server-Side Request Forgery)]]
📚 Fuente: [PayloadsAllTheThings — Web Cache Deception](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Web%20Cache%20Deception) (MIT, © Swissky)
