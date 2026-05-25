---
tags:
  - seguridad-web
  - owasp
  - vhost
aliases:
  - Virtual Hosts (vhost enumeration)
  - Virtual Hosts
  - Vhost enumeration
  - Enumeración de vhosts
---

# Virtual Hosts (vhost enumeration)

**La enumeración de virtual hosts busca sitios web "ocultos" servidos por una misma IP que se distinguen únicamente por la cabecera `Host`, revelando aplicaciones internas, entornos de staging o paneles que no aparecen en el DNS público.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Un servidor (Apache, Nginx) puede alojar muchos sitios en la misma IP usando *name-based virtual hosting*: elige qué contenido servir según la cabecera `Host` de la petición. Si un vhost no tiene registro DNS público (`dev.empresa.com`, `admin.interno`), no se encuentra por subdominio normal, pero **sí** respondiendo a la IP con el `Host` correcto. La diferencia con la enumeración de subdominios es clave: aquí se fuerza la cabecera contra una IP fija en vez de resolver nombres por DNS.

## Detección y pruebas
Fija la IP objetivo y haz fuzzing de la cabecera `Host` con un diccionario, comparando longitud, código de estado y hash de la respuesta frente a la respuesta "por defecto". Un vhost válido suele devolver contenido distinto. Combínalo con reconocimiento previo (certificados TLS/SAN, DNS pasivo) para nutrir el diccionario.

## Explotación / payloads
**Prueba manual con curl** (forzar Host sobre una IP):

```bash
curl -s -H "Host: dev.empresa.com" http://10.10.10.5/ -o /dev/null -w "%{http_code} %{size_download}\n"
curl -s -H "Host: admin.empresa.com" http://10.10.10.5/
```

**Fuzzing con ffuf** (FUZZ en la cabecera Host):

```bash
ffuf -u http://10.10.10.5/ -H "Host: FUZZ.empresa.com" \
     -w subdominios.txt -fs 1234     # filtra por tamaño de la respuesta base
```

**Con gobuster (modo vhost):**

```bash
gobuster vhost -u http://10.10.10.5 -w subdominios.txt --append-domain
```

## Mitigación
- Configurar un **vhost por defecto** que devuelva 403/404 ante un `Host` desconocido (`default_server` en Nginx).
- No exponer entornos internos/staging en servidores accesibles desde Internet; segmentar por red/VPN.
- Validar la cabecera `Host` contra una lista de dominios permitidos en la aplicación.
- Evitar filtrar nombres internos en certificados TLS y respuestas de error.

## Herramientas
- **ffuf**, **gobuster (vhost)**, **wfuzz** — fuzzing de la cabecera Host.
- **VHostScan** — enumeración específica de virtual hosts.

---
🔗 Relacionado: [[Reverse Proxy mal configurado]] · [[SSRF (Server-Side Request Forgery)]] · [[Fuzzing de Directorios y Subdominios]]
📚 Fuente: [PayloadsAllTheThings — Virtual Hosts](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Virtual%20Hosts) (MIT, © Swissky)
