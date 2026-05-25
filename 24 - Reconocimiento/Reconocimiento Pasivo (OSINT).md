---
tags:
  - reconocimiento
  - osint
  - pentesting
  - recon-pasivo
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/methodology/bug-hunting-methodology/"
licencia: MIT
---

# Reconocimiento Pasivo (OSINT)

> [!warning] Uso ético y autorizado
> Aplicar solo en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

El reconocimiento pasivo recopila información sin interactuar directamente con el objetivo. No genera tráfico hacia el target.

---

## Motores de búsqueda de dispositivos expuestos

| Herramienta | URL | Uso |
|-------------|-----|-----|
| Shodan | shodan.io | Puertos, banners, servicios expuestos |
| FOFA | fofa.info | Similar a Shodan, base china |
| ZoomEye | zoomeye.ai | Similar a Shodan |
| Odin | search.odin.io | Alternativa moderna |

```bash
# Integración Shodan con Nmap
nmap --script shodan-hq.nse --script-args 'apikey=<API_KEY>,target=<TARGET>'
```

### Buscar por favicon (detectar apps detrás de CDN)
```bash
# fav-up: busca el hash del favicon en Shodan
python3 favUp.py --favicon-url https://target.com/favicon.ico -sc
python3 favUp.py --web target.com -s
```

---

## Wayback Machine (endpoints olvidados)

```bash
# Buscar JS files y endpoints históricos
curl -sX GET "http://web.archive.org/cdx/search/cdx?url=target.com&output=text&fl=original&collapse=urlkey&matchType=prefix"

# Con waybackurls (automatizado)
echo "target.com" | waybackurls

# Con gau (URLs de múltiples fuentes: Wayback, Common Crawl, OTX)
gau --o target-urls.txt target.com
gau --blacklist png,jpg,gif,css target.com
```

---

## theHarvester (emails, subdominios, IPs, empleados)

```bash
# Búsqueda en todas las fuentes disponibles
python theHarvester.py -b all -d target.com

# Fuentes específicas
python theHarvester.py -b google,bing,linkedin -d target.com
```

---

## Google Dorks

```bash
# Subdominios expuestos
site:*.target.com -www

# Archivos de configuración indexados
intext:"dhcpd.conf" "index of"

# Paneles de login
intitle:"SSL Network Extender Login" -checkpoint.com

# Archivos sensibles
site:target.com ext:pdf OR ext:xls OR ext:doc
site:target.com inurl:admin
site:target.com intitle:"index of"
```

---

## GitHub OSINT (secretos en repos públicos)

```bash
# gitrob: busca tokens, claves, credenciales en repos de una organización
gitrob analyze <usuario_o_org> \
  --site=https://github.com \
  --endpoint=https://api.github.com \
  --access-tokens=<token>
```

Buscar manualmente en GitHub:
```
org:target.com password
org:target.com secret_key
org:target.com api_key
org:target.com "BEGIN RSA PRIVATE KEY"
```

---

## Enumeración de subdominios (pasiva)

```bash
# HackerTarget (sin autenticación)
curl --silent 'https://api.hackertarget.com/hostsearch/?q=target.com' | grep -o '\w.*target.com'

# CommonCrawl
echo "target.com" | xargs -I domain curl -s \
  "http://index.commoncrawl.org/CC-MAIN-2024-index?url=*.target.com&output=json" \
  | jq -r .url | sort -u

# Certificate Transparency Logs (crt.sh)
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u
```

---

## Buckets S3 públicos

- [buckets.grayhatwarfare.com](https://buckets.grayhatwarfare.com/) — búsqueda de buckets expuestos
- [shorteners.grayhatwarfare.com](https://shorteners.grayhatwarfare.com/) — URLs acortadas con información

```bash
# urlhunter: busca el dominio en URLs acortadas indexadas
urlhunter --keywords keywords.txt --date 2024-01-01
```

---

## Flujo de trabajo recomendado

```
1. Shodan/FOFA        → Servicios expuestos, IPs del objetivo
2. crt.sh             → Subdominios por Certificate Transparency
3. theHarvester       → Emails, nombres, más subdominios
4. Google Dorks       → Archivos sensibles, paneles, configs
5. GitHub OSINT       → Secretos y credenciales filtradas
6. Wayback Machine    → Endpoints olvidados, JS antiguo
7. GrayHatWarfare     → Buckets públicos
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Reconocimiento Activo de Red]]
- [[Reconocimiento Web]]
- Fuente: [InternalAllTheThings - Bug Hunting Methodology](https://swisskyrepo.github.io/InternalAllTheThings/methodology/bug-hunting-methodology/) — MIT
