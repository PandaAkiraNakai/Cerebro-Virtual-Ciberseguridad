---
tags:
  - seguridad-web
  - owasp
  - randomness
aliases:
  - Aleatoriedad insegura
  - Insecure Randomness
---

# Aleatoriedad insegura

**La aleatoriedad insegura aparece cuando un valor que debería ser impredecible (token, identificador, contraseña temporal) se genera con una fuente débil o predecible.** Si el atacante puede reproducir o adivinar el valor, logra acceso no autorizado o suplantación.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Muchos generadores de números pseudoaleatorios (PRNG) no son criptográficamente seguros o se inicializan con una semilla predecible (la hora del sistema). Si conoces o estimas la semilla, puedes regenerar la secuencia completa. Lo mismo ocurre con identificadores que incrustan información temporal o secuencial: GUID v1, ObjectId de MongoDB o `uniqid()` de PHP no son aleatorios reales, sino derivados de tiempo, contador, máquina y proceso.

## Detección y pruebas
Recolecta varios tokens consecutivos (registro, recuperación de contraseña, sesión) y analiza su estructura:
- ¿La parte variable crece con el tiempo o de forma secuencial?
- ¿El formato corresponde a un GUID v1, un ObjectId o un `uniqid`?

```
xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx   # M = versión del UUID
```

| Versión UUID | Base |
|---|---|
| 1 | tiempo + secuencia de reloj (predecible) |
| 3 / 5 | hash MD5 / SHA1 |
| 4 | aleatorio (preferible) |

## Explotación / payloads
**Semilla basada en tiempo:** si conoces el instante de generación, reproduces el valor.

```python
import random, time
seed = int(time.mktime(time.strptime('2024-11-10 13:37', '%Y-%m-%d %H:%M')))
random.seed(seed)
print(random.randint(1, 100))   # mismo resultado que la víctima
```

**Identificadores predecibles** (revertir un `uniqid` de PHP al timestamp original):

```python
def reverse_uniqid(value):
    sec = int(value[:8], 16); usec = int(value[8:], 16)
    return float(f"{sec}.{usec}")
```

**Ataque sandwich:** captura un token justo antes y otro justo después de la acción objetivo para acotar el rango temporal y predecir el token intermedio (recuperación de contraseña, invitaciones). El ObjectId de MongoDB se predice de forma equivalente por su timestamp y contador.

## Mitigación
- Usar generadores criptográficamente seguros (`secrets` en Python, `random_bytes()`/`/dev/urandom`, `crypto.randomBytes` en Node).
- Generar tokens y secretos con suficiente entropía (>=128 bits) y formato no derivado de tiempo (UUID v4 aleatorio).
- No reutilizar el PRNG general (`rand`, `mt_rand`, `random`) para fines de seguridad ni implementar algoritmos propios.
- Caducar tokens rápidamente, atarlos a la sesión y limitar intentos para frenar la predicción.

## Herramientas
- **guidtool** — inspecciona y ataca GUID v1.
- **mongo-objectid-predict** — predice ObjectId de MongoDB.
- **mt_rand-reverse** (ambionics) — recupera la semilla de `mt_rand()` con dos salidas.
- **reset-tolkien** — detección y ataque sandwich de secretos basados en tiempo.

---
🔗 Relacionado: [[IDOR]] · [[Fuerza bruta y rate limiting]] · [[Type Juggling (PHP)]]
📚 Fuente: [PayloadsAllTheThings — Insecure Randomness](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Randomness) (MIT, © Swissky)
