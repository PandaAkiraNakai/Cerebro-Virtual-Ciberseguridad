---
tags:
  - seguridad-web
  - owasp
  - hidden-parameters
aliases:
  - Parámetros ocultos
  - Hidden Parameters
  - Parámetros no documentados
---

# Parámetros ocultos

**Los parámetros ocultos son argumentos que la aplicación procesa pero que no aparecen en la interfaz ni en la documentación (restos de depuración, funciones internas, flags). Descubrirlos amplía la superficie de ataque y a menudo destapa funcionalidades privilegiadas o inseguras.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El backend suele aceptar más parámetros de los que el front-end envía: `debug`, `admin`, `test`, `source`, `preview`, `role`. Quedan ahí por desarrollo, compatibilidad o lógica condicional olvidada. Si el atacante los adivina y los incluye, puede activar modos de depuración (que filtran datos), saltarse controles o alcanzar rutas internas. Es un paso de reconocimiento que potencia [[Mass Assignment]], [[IDOR]] y [[HTTP Parameter Pollution (HPP)]].

## Detección y pruebas
Fuzzing de nombres de parámetro contra el endpoint, comparando longitud/código/tiempo de respuesta frente a la base. Diferencia GET (query string) y POST (body, JSON). Fuentes para el diccionario: el propio JavaScript del sitio, archivos `.map`, parámetros vistos en otras rutas y wordlists especializadas. Una respuesta que cambia al añadir un parámetro indica que se procesa.

## Explotación / payloads
**Fuzzing con Arjun:**

```bash
arjun -u https://victima.com/api/perfil
arjun -u https://victima.com/api/perfil -m POST --headers "Content-Type: application/json"
```

**Fuzzing con Param Miner** (Burp) — inyecta nombres y detecta diferencias automáticamente.

**Parámetros que suelen valer la pena probar:**

```
debug=true   test=1   admin=1   role=admin   source=true
preview=1    edit=1   internal=1   raw=1   show_errors=1
callback=    redirect=   next=   url=   file=
```

**Extracción desde el front-end** (parámetros referenciados en el JS):

```bash
# buscar nombres candidatos en los bundles
curl -s https://victima.com/app.js | grep -oE "[?&][a-zA-Z_]+=" | sort -u
```

## Mitigación
- Ignorar y registrar parámetros no esperados; no dejar flags de depuración accesibles en producción.
- Definir esquemas estrictos de entrada (allowlist de parámetros) por endpoint.
- Eliminar código y rutas de prueba antes de desplegar; revisar diffs por parámetros condicionales.
- No exponer source maps ni comentarios que delaten parámetros internos.

## Herramientas
- **Arjun** — descubrimiento de parámetros HTTP.
- **Param Miner** (Burp), **ffuf** (`FUZZ` como nombre de parámetro), **x8**.

---
🔗 Relacionado: [[Mass Assignment]] · [[HTTP Parameter Pollution (HPP)]] · [[IDOR]] · [[ORM Leak]]
📚 Fuente: [PayloadsAllTheThings — Hidden Parameters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Hidden%20Parameters) (MIT, © Swissky)
