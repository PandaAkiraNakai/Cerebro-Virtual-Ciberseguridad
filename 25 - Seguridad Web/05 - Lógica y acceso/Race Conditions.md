---
tags:
  - seguridad-web
  - owasp
  - race-condition
aliases:
  - Race Conditions
  - Condiciones de carrera
  - Race Condition
---

# Race Conditions

**Una condición de carrera ocurre cuando el resultado depende del orden o el instante en que se procesan varias peticiones concurrentes sobre un recurso compartido.** Enviando solicitudes casi simultáneas se gana una "ventana" en la que las comprobaciones aún no se han aplicado, rompiendo límites de negocio.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Entre que la aplicación lee un estado (saldo, cupón, intentos) y lo actualiza, existe un breve intervalo. Si llegan varias peticiones a la vez, todas leen el mismo estado antiguo y operan sobre él antes de que se persista el primer cambio. Dos patrones típicos:
- **Limit-overrun:** superar un límite (canjear un cupón varias veces, exceder un saldo, votar de más).
- **Rate-limit bypass:** saltar protecciones de fuerza bruta o 2FA porque el contador no se sincroniza.

## Detección y pruebas
Identifica acciones que deberían ser únicas o limitadas (canjear, aplicar descuento, registrar, validar OTP). Captura la petición y reenvíala muchas veces en paralelo; si el efecto se aplica más veces de lo permitido, hay condición de carrera. La clave es minimizar el jitter de red para que las solicitudes lleguen juntas.

## Explotación / payloads
**HTTP/2 single-packet attack:** ~20-30 peticiones en un único paquete llegan simultáneamente, eliminando el jitter. En Burp: duplicar la petición en Repeater, agruparlas y "Send group in parallel".

**HTTP/1.1 last-byte sync:** se envían todas las peticiones menos el último byte y luego se "libera" ese byte a la vez.

**Turbo Intruder** (sincronización con compuerta):

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=100,
                           pipeline=False)
    for i in range(30):
        engine.queue(target.req, gate='race1')
    engine.openGate('race1')
    engine.complete(timeout=60)
```

## Mitigación
- Usar transacciones atómicas y bloqueos a nivel de base de datos (`SELECT ... FOR UPDATE`, restricciones únicas).
- Aplicar operaciones idempotentes y claves de idempotencia en acciones sensibles.
- No delegar la concurrencia al framework por defecto; serializar las secciones críticas.
- Reforzar rate limiting y bloqueos de cuenta de forma que el contador se actualice de forma atómica.

## Herramientas
- **Turbo Intruder** (extensión de Burp) — envío masivo de peticiones sincronizadas.
- **Raceocat** — explotación de condiciones de carrera en webs.
- **h2spacex** — librería para single-packet attack en HTTP/2.

---
🔗 Relacionado: [[Fuerza bruta y rate limiting]] · [[Errores de lógica de negocio]] · [[IDOR]]
📚 Fuente: [PayloadsAllTheThings — Race Condition](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Race%20Condition) (MIT, © Swissky)
