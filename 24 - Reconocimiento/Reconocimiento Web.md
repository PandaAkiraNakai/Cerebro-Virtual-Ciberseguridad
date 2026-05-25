---
tags:
  - reconocimiento
  - web
  - subdominios
  - pentesting
  - recon-activo
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/methodology/bug-hunting-methodology/"
licencia: MIT
---

# Reconocimiento Web

> [!warning] Uso ético y autorizado
> Aplicar solo en sistemas propios o con autorización. Ver [[📜 Fuentes y Licencias]].

---

## Enumeración de subdominios

### Herramientas pasivas
```bash
# subfinder (fuentes OSINT: VirusTotal, Shodan, etc.)
subfinder -d target.com

# amass (pasivo)
amass enum -passive -dir /tmp/amass/ -d target.com -o subdominios.txt

# crt.sh (Certificate Transparency)
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u
```

### Herramientas activas (fuerza bruta)
```bash
# altdns: permutaciones de subdominios ya encontrados
altdns -i subdominios.txt -o permutaciones.txt -w words.txt

# gotator: genera más permutaciones
gotator -sub subdominios.txt -perm permutaciones.txt

# massdns: resuelve subdominios a IPs (alta velocidad)
massdns -r resolvers.txt -o S -w massdns.out subdominios.txt
```

### Subdomain Takeover
```bash
# Verificar si un subdominio apunta a servicio sin reclamar
# Lista de servicios vulnerables: github.com/EdOverflow/can-i-take-over-xyz
```

---

## Fuzzing de directorios y archivos

```bash
# ffuf
ffuf -H 'User-Agent: Mozilla' -v -t 30 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u 'https://target.com/FUZZ'

# gobuster
gobuster dir -a 'Mozilla' -e -k -l -t 30 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u 'https://target.com/'

# Con cookies de sesión autenticada
ffuf -u 'https://target.com/FUZZ' -w wordlist.txt \
  -b 'SESSION=abc123' -mc 200,301,302,403
```

> [!tip] Wordlists especializadas
> Usar listas de [wordlists.assetnote.io](https://wordlists.assetnote.io) por tecnología detectada:
> - `httparchive_php_2026_01_27.txt` para PHP
> - `httparchive_directories_1m.txt` para directorios generales

### Archivos de backup y temporales
```bash
# bfac: detecta .bak, .old, .orig, .tmp, ~, etc.
bfac --url http://target.com/index.php --level 4
bfac --list urls.txt
```

---

## Crawling y descubrimiento de URLs

```bash
# katana: crawler moderno (JS-aware)
katana -u https://target.com

# hakrawler: rápido y simple
echo https://target.com | hakrawler

# gau: URLs históricas (Wayback + CommonCrawl + OTX)
gau --o target-urls.txt target.com
gau --blacklist png,jpg,gif,css target.com

# waybackurls: solo Wayback Machine
echo "target.com" | waybackurls
```

---

## Archivos especiales a revisar siempre

| Archivo | Qué buscar |
|---------|-----------|
| `robots.txt` | Rutas ocultas en `Disallow:` |
| `sitemap.xml` | Mapa completo de URLs |
| `security.txt` | Contacto de seguridad |
| `.git/HEAD` | Repo git expuesto (`git clone http://target.com/.git`) |
| `phpinfo.php` | Configuración PHP completa |
| `/.env` | Variables de entorno con credenciales |
| `/backup.zip`, `/backup.tar.gz` | Backups del código |

---

## Detección de tecnologías

```bash
# httpx: tech-detect, favicon hash, JARM, ASN, status
httpx -title -tech-detect -status-code -follow-redirects \
  -jarm -asn -json -silent -ports 80,443 -l urls.txt

# cdncheck: detectar WAF / CDN
echo www.target.com | cdncheck -resp
# Output: www.target.com [waf] [cloudflare]
```

---

## Parámetros ocultos

```bash
# arjun: descubre parámetros GET/POST no documentados
arjun -u https://target.com/endpoint

# x8
x8 -u "https://target.com/?known=1" -w params-wordlist.txt
```

---

## Endpoints Next.js

En aplicaciones Next.js, el manifest del build expone todas las rutas:

```javascript
// En DevTools → Console
console.log(window.__BUILD_MANIFEST)
console.log(__BUILD_MANIFEST.sortedPages)
```

---

## Comentarios en HTML y JS

```bash
# Buscar comentarios y TODOs en el código fuente
curl -s https://target.com | grep -oE '<!--.*?-->'
curl -s https://target.com/app.js | grep -i 'todo\|fixme\|secret\|password\|api'
```

---

## Escaneo automático de vulnerabilidades

```bash
# nuclei: templates para cientos de CVEs y misconfigs
nuclei -u https://target.com
nuclei -u https://target.com -t technologies/ -t exposures/

# nikto: scanner web clásico
nikto -h http://target.com
```

---

## Proxies para testing manual

| Herramienta | Uso |
|-------------|-----|
| Burp Suite Community | Intercepción, repeater, scanner |
| OWASP ZAP | Alternativa open source a Burp |
| Caido | Proxy moderno y ligero |

---

## Flujo de trabajo recomendado

```
1. subfinder / amass      → Subdominios del objetivo
2. httpx                   → Filtrar los que responden + tech
3. katana / gau            → URLs y endpoints de cada subdominio
4. ffuf / gobuster         → Fuzzing de directorios
5. arjun                   → Parámetros ocultos
6. nuclei / nikto          → Vulnerabilidades automáticas
7. Burp Suite              → Testing manual y correlación
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Reconocimiento Pasivo (OSINT)]]
- [[Reconocimiento Activo de Red]]
- [[Fuzzing de Directorios y Subdominios]]
- Fuente: [InternalAllTheThings - Bug Hunting Methodology](https://swisskyrepo.github.io/InternalAllTheThings/methodology/bug-hunting-methodology/) — MIT
