---
tags:
  - seguridad-web
  - owasp
  - open-redirect
aliases:
  - Open Redirect
  - Redirección abierta
---

# Open Redirect

**La redirección abierta (Open Redirect)** ocurre cuando una aplicación redirige al usuario a una URL tomada de una entrada no validada, permitiendo al atacante enviarlo a un sitio malicioso.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
La aplicación recibe una URL del usuario (normalmente en un parámetro) y la usa como destino de un redirect sin comprobar que apunte a su propio dominio. El atacante reemplaza ese destino por un sitio que controla:

```
https://example.com/redirect?url=https://atacante.example
```

Como el dominio visible al inicio del enlace es el legítimo, resulta convincente para **phishing**, robo de credenciales o de tokens (OAuth `redirect_uri`). También sirve para encadenar con SSRF o saltarse controles de acceso.

## Detección y pruebas
- Localiza parámetros candidatos: `?url=`, `?next=`, `?redirect=`, `?dest=`, `?returnTo=`, `?continue=`, `?rurl=`, `?checkout_url=`, etc.
- Cambia el valor por un dominio externo controlado y observa si la respuesta devuelve `Location:` apuntando ahí (códigos **3xx**: 301, 302, 303, 307, 308).
- Prueba también redirecciones **basadas en path** (`/redirect/http://...`) y **en JavaScript** (`window.location = valorUsuario`, redirección DOM-based).

## Explotación / payloads

**Parámetros y rutas comunes:**

```
?url={payload}   ?next={payload}   ?redirect={payload}   ?dest={payload}
?returnTo={payload}   ?continue={payload}   /{payload}   /redirect/{payload}
```

**Bypass de filtros (lista negra / validación de dominio):**

```
https://whitelisted.com.atacante.example/   # dominio permitido como subdominio
//atacante.example                           # esquema relativo evade filtro "http"
////atacante.example
https:atacante.example                       # evade filtro "//"
\/\/atacante.example/                        # backslashes
https://example.com@atacante.example/        # "@" : el host real es el de la derecha
https://example.com?atacante.example         # "?" tratado por el navegador como "/?"
//atacante%00.example                         # null byte
?next=example.com&next=atacante.example       # HTTP Parameter Pollution
//atacante%E3%80%82example                    # "。" (U+3002) en vez de "."
```

> Normalización Unicode: caracteres como `℀` o `／` pueden colapsar a `/` o `.` tras normalizar, cambiando el host efectivo.

## Mitigación
- **No uses entrada del usuario como URL de destino.** Si es imprescindible, valida contra una **allowlist** de rutas/dominios permitidos.
- Prefiere redirecciones **relativas** o un mapa de IDs → URL en servidor (el usuario envía un identificador, no una URL).
- Rechaza valores que empiecen por `//`, `\\`, esquemas absolutos o que contengan `@`, y normaliza antes de validar.
- En OAuth, exige coincidencia exacta del `redirect_uri` registrado.
- Muestra una página intermedia de aviso al salir a dominios externos.

## Herramientas
- **Open-Redirect-Payloads** (Predrag Cujanović) — listas de payloads.
- Burp Suite / fuzzers para inyectar payloads en parámetros candidatos.

---
🔗 Relacionado: [[SSRF (Server-Side Request Forgery)]] · [[HTTP Request Smuggling]] · [[Web Cache Deception]]
📚 Fuente: [PayloadsAllTheThings — Open Redirect](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Open%20Redirect) (MIT, © Swissky)
