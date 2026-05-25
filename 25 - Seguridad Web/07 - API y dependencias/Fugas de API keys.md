---
tags:
  - seguridad-web
  - owasp
  - api-keys
aliases:
  - Fugas de API keys
  - API Key Leaks
  - Secrets leaks
  - Fuga de secretos
---

# Fugas de API keys

**Las fugas de API keys ocurren cuando claves, tokens o secretos quedan expuestos en lugares accesibles —repositorios, bundles de JavaScript, apps móviles, historiales, logs— permitiendo a un atacante usar servicios de pago, acceder a datos o pivotar hacia infraestructura cloud.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Los secretos terminan donde no deberían: hardcodeados en el front-end, commiteados a Git (y persistentes en el historial aunque se borren), incrustados en APKs, en variables de error, en parámetros de URL que acaban en logs, o en endpoints de [[GraphQL (ataques)]] mal protegidos. Una vez obtenida, el impacto depende del **alcance** de la clave: desde gasto en una API hasta acceso completo si es una clave cloud (AWS, GCP) sin restricciones.

## Detección y pruebas
Escanea el JavaScript público, los archivos `.map`, los repos (incluido el historial), las apps móviles descompiladas y las respuestas de error. Busca patrones conocidos (prefijos `AKIA`, `sk_live_`, `ghp_`, `AIza`, JWTs). Tras encontrar una clave, **verifica su validez y permisos** de forma no destructiva contra el endpoint correspondiente.

## Explotación / payloads
**Búsqueda de secretos en bundles y disco:**

```bash
trufflehog git https://github.com/org/repo --only-verified
gitleaks detect --source . -v
curl -s https://victima.com/app.js | grep -oE "(AKIA[0-9A-Z]{16}|sk_live_[0-9a-zA-Z]{24}|AIza[0-9A-Za-z_-]{35}|ghp_[0-9A-Za-z]{36})"
```

**Patrones frecuentes:**

```
AWS Access Key      AKIA....              (par con Secret de 40 chars)
Stripe (live)       sk_live_....
Google API          AIza....
GitHub PAT          ghp_....
Slack token         xox[baprs]-....
JWT                 eyJ....eyJ....sig
```

**Validación del alcance** (ejemplo AWS, solo lectura de identidad):

```bash
AWS_ACCESS_KEY_ID=... AWS_SECRET_ACCESS_KEY=... aws sts get-caller-identity
```

## Mitigación
- **Nunca** poner secretos en el cliente ni en el repo; usar gestores de secretos (Vault, AWS Secrets Manager) y variables de entorno del servidor.
- Escanear commits con hooks (gitleaks/trufflehog) en CI; purgar el historial si ya se filtró y **rotar** la clave de inmediato.
- Claves con privilegio mínimo, restringidas por IP/dominio/scope y con caducidad; monitorizar uso anómalo.
- No registrar secretos en logs ni pasarlos por query string.

## Herramientas
- **TruffleHog**, **Gitleaks** — detección de secretos en repos e historial.
- **Nuclei** (plantillas de exposed-tokens), **SecretFinder** — secretos en JS.

---
🔗 Relacionado: [[GraphQL (ataques)]] · [[SSRF (Server-Side Request Forgery)]] · [[JWT (JSON Web Token)]]
📚 Fuente: [PayloadsAllTheThings — API Key Leaks](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/API%20Key%20Leaks) (MIT, © Swissky)
