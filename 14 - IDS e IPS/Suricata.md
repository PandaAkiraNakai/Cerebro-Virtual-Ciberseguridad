---
tags:
  - ids
  - ips
  - suricata
  - linux
  - seguridad
aliases:
  - Suricata
  - Suricata IDS
  - Suricata IPS
  - suricata.yaml
  - suricata-update
---

# Suricata

**Suricata** es un sistema de detección y prevención de intrusiones (**IDS/IPS**) de código abierto, mantenido por la OISF. Inspecciona el tráfico en tiempo real, lo compara con reglas y genera alertas; en modo IPS además bloquea.

## Cómo funciona

| Etapa | Qué hace |
|---|---|
| Captura | Escucha el tráfico de una interfaz (AF_PACKET, pcap o NFQUEUE) |
| Decodificación | Separa IP, puertos y protocolos |
| Inspección profunda (DPI) | Reconoce aplicaciones (HTTP, TLS, DNS, SSH…) y su contenido |
| Comparación | Contrasta el tráfico con las reglas cargadas |
| Alerta | Emite el evento con todos sus detalles |
| Registro | Guarda alertas, flujos y metadatos para análisis posterior |
| Prevención (opcional) | Como IPS, descarta el tráfico malicioso |

## Instalación

```bash
sudo apt update && sudo apt upgrade

# Debian y Kali
sudo apt install suricata jq

# Ubuntu: repositorio oficial con la versión estable más reciente
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq

suricata --build-info | head     # versión y capacidades compiladas
```

Descarga las reglas **ET Open** (sin reglas, Suricata no detecta nada):

```bash
sudo suricata-update
```

Quedan en `/var/lib/suricata/rules/suricata.rules`.

## Archivos clave

| Ruta | Contenido |
|---|---|
| `/etc/suricata/suricata.yaml` | Configuración principal |
| `/etc/default/suricata` | Interfaz y modo que usa el servicio en Debian y Ubuntu (`IFACE=`, `LISTENMODE=`) |
| `/var/lib/suricata/rules/suricata.rules` | Reglas generadas por `suricata-update` |
| `/etc/suricata/rules/local.rules` | Tus reglas propias (lo creas tú) |
| `/var/log/suricata/` | Logs |

## Configuración de suricata.yaml

```bash
sudo nano /etc/suricata/suricata.yaml
```

| Parámetro | Qué cambiar | Ejemplo |
|---|---|---|
| `HOME_NET` | Red que proteges | `HOME_NET: "[192.168.1.0/24]"` |
| `af-packet` → `interface` | Interfaz de captura | `- interface: ens33` |
| `pcap` → `interface` | Interfaz si usas el modo pcap | `- interface: ens33` |
| `community-id` | Identificador común de flujo para correlacionar en un SIEM | `community-id: true` |
| `rule-files` | Reglas que se cargan | Agrega tu `local.rules` |

Extracto de las secciones que tocas:

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.1.0/24]"
    EXTERNAL_NET: "!$HOME_NET"

af-packet:
  - interface: ens33
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes

default-rule-path: /var/lib/suricata/rules
rule-files:
  - suricata.rules
  - /etc/suricata/rules/local.rules    # ruta absoluta para tus reglas

outputs:
  - eve-log:
      enabled: yes
      filename: eve.json
      community-id: true
```

En Debian y Ubuntu, ajusta también la interfaz del servicio:

```bash
sudo sed -i 's/^IFACE=.*/IFACE=ens33/' /etc/default/suricata
```

> [!tip] Averigua tu interfaz
> `ip -br link` lista las interfaces. En una VM suele ser `ens33`, `enp0s3` o `eth0`.

## Validar y arrancar

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v     # valida configuración y reglas

sudo systemctl enable --now suricata                   # como servicio
sudo systemctl restart suricata
sudo systemctl status suricata

# En primer plano, útil para depurar
sudo suricata -c /etc/suricata/suricata.yaml -i ens33
# Con un archivo de reglas adicional
sudo suricata -c /etc/suricata/suricata.yaml -s /etc/suricata/rules/local.rules -i ens33
# Analizar una captura en vez de tráfico en vivo
sudo suricata -c /etc/suricata/suricata.yaml -r captura.pcap -l /tmp/salida-suricata
```

Sigue las alertas en vivo en otra terminal:

```bash
sudo tail -f /var/log/suricata/fast.log
```

## Prueba rápida de detección

Desde un equipo de la red monitoreada:

```bash
curl http://testmynids.org/uid/index.html
```

Debe aparecer en `fast.log` la alerta **GPL ATTACK_RESPONSE id check returned root** (sid 2100498). Si sale, captura, reglas y logs funcionan.

## Reglas propias

Anatomía de una regla:

```text
acción  protocolo  origen puerto  ->  destino puerto  (opciones;)
alert   tcp        any    any     ->  $HOME_NET 22    (msg:"..."; sid:1000001; rev:1;)
```

| Acción | Efecto |
|---|---|
| `alert` | Genera alerta |
| `pass` | Deja pasar y deja de inspeccionar |
| `drop` | Descarta y alerta (solo en modo IPS) |
| `reject` | Descarta y envía RST o ICMP de rechazo |

Usa `sid` desde 1000000 para reglas locales y sube `rev` en cada cambio.

```bash
sudo nano /etc/suricata/rules/local.rules
```

```text
# Ping entrante
alert icmp any any -> $HOME_NET any (msg:"LOCAL ICMP ping detectado"; itype:8; sid:1000001; rev:1;)

# Más de 5 intentos de conexión SSH en 60 s desde el mismo origen
alert tcp any any -> $HOME_NET 22 (msg:"LOCAL posible fuerza bruta SSH"; flow:to_server; flags:S; threshold:type both, track by_src, count 5, seconds 60; sid:1000002; rev:1;)

# Escaneo SYN: 20 SYN en 10 s desde el mismo origen
alert tcp any any -> $HOME_NET any (msg:"LOCAL posible escaneo SYN"; flags:S,12; threshold:type both, track by_src, count 20, seconds 10; sid:1000003; rev:1;)

# Path traversal en la URI
alert http any any -> $HOME_NET any (msg:"LOCAL path traversal en URI"; flow:established,to_server; http.uri; content:"../"; sid:1000004; rev:1;)

# Consulta DNS a un dominio vigilado
alert dns any any -> any any (msg:"LOCAL consulta a dominio vigilado"; dns.query; content:"dominio-malicioso.test"; nocase; sid:1000005; rev:1;)
```

Después de editar: valida con `suricata -T` y recarga.

## Mantener las reglas

```bash
sudo suricata-update                          # actualizar (prográmalo con cron o un timer)
sudo suricata-update list-sources             # fuentes disponibles
sudo suricata-update enable-source oisf/trafficid
sudo suricata-update update-sources

# Recargar reglas sin reiniciar
sudo suricatasc -c reload-rules
```

Para silenciar una regla ruidosa, pon su sid en `/etc/suricata/disable.conf` (una línea por sid) y vuelve a correr `suricata-update`.

| Fuente | Descripción |
|---|---|
| ET Open | Catálogo gratuito de Proofpoint; viene activado por defecto |
| ET Pro | Comercial: más reglas, soporte e inteligencia exclusiva |
| Otras | Fuentes de la OISF y repositorios por industria; `list-sources` las muestra |

Descarga manual de ET Open (alternativa a `suricata-update`):

```bash
wget https://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
sudo tar -xzf emerging.rules.tar.gz -C /etc/suricata/
# agrega los .rules que quieras en rule-files de /etc/suricata/suricata.yaml
sudo systemctl restart suricata
```

`suricata-update` es la herramienta oficial; Oinkmaster y PulledPork vienen del mundo de [[Snort]].

## Modo IPS con NFQUEUE

En un Linux que enruta tráfico, iptables manda los paquetes a una cola y Suricata decide si pasan.

```bash
# Router o gateway: tráfico que atraviesa el equipo
sudo iptables -I FORWARD -j NFQUEUE --queue-num 0 --queue-bypass
# Host: tráfico propio
sudo iptables -I INPUT  -j NFQUEUE --queue-num 0 --queue-bypass
sudo iptables -I OUTPUT -j NFQUEUE --queue-num 0 --queue-bypass

sudo suricata -c /etc/suricata/suricata.yaml -q 0
```

Cambia `alert` por `drop` en las reglas que deben bloquear. `--queue-bypass` deja pasar el tráfico si Suricata no está corriendo, para no cortar la red. Ver [[iptables]].

> [!warning] Prueba en IDS antes de bloquear
> Una regla `drop` con falsos positivos corta servicios legítimos. Déjala en `alert` unos días y revisa qué dispara.

## Logs

| Archivo | Contenido | Uso |
|---|---|---|
| `eve.json` | Alertas, flujos, DNS, HTTP, TLS, archivos, en JSON | Análisis forense y envío a un SIEM |
| `fast.log` | Una línea por alerta: hora, origen y destino, regla, clasificación y prioridad | Revisión rápida |
| `stats.log` | Paquetes procesados, descartes, memoria | Rendimiento |
| `suricata.log` | Arranque, errores y advertencias | Resolver problemas |

```bash
# Alertas legibles
jq -c 'select(.event_type=="alert") | {t:.timestamp, src:.src_ip, dst:.dest_ip, sig:.alert.signature}' /var/log/suricata/eve.json

# Firmas que más disparan
jq -r 'select(.event_type=="alert") | .alert.signature' /var/log/suricata/eve.json | sort | uniq -c | sort -rn | head

# Consultas DNS observadas
jq -r 'select(.event_type=="dns") | .dns.rrname // empty' /var/log/suricata/eve.json | sort | uniq -c | sort -rn | head
```

Para centralizar, envía `eve.json` a [[Splunk Enterprise]] o a otro SIEM; `community-id` permite cruzarlo con otros sensores.

## Problemas y rendimiento

> [!warning] "fanout not supported by kernel"
> La función *fanout* de AF_PACKET, que reparte el procesamiento entre núcleos, no está disponible.
> - **Kernel antiguo:** revisa con `uname -r` y actualiza el kernel.
> - **`cluster-id` ocupado:** otra instancia lo está usando. Revisa `ps aux | grep suricata` y cambia `cluster-id` (por ejemplo, de 99 a 100).
> - **`cluster-type`:** prueba otro modo y ajusta la afinidad de CPU.

| `cluster-type` | Descripción | Ideal para |
|---|---|---|
| `cluster_flow` | Agrupa paquetes por flujo; mejor localidad de datos | La mayoría de los sistemas |
| `cluster_cpu` | Reparte secuencialmente entre núcleos | NIC sin RSS; requiere afinidad |
| `cluster_qm` | Como `cluster_cpu`, con colas MPMC | Muchos núcleos; requiere afinidad |

| Síntoma | Revisión |
|---|---|
| No aparecen alertas | Interfaz equivocada en `suricata.yaml` o en `/etc/default/suricata`, `HOME_NET` mal definido o reglas sin descargar |
| `0 rules successfully loaded` | Falta correr `suricata-update` o `rule-files` apunta mal |
| Paquetes descartados en `stats.log` | Más hilos, `cluster_flow`, desactivar reglas que no aplican |
| Muchos falsos positivos | Ajusta `HOME_NET`, silencia con `disable.conf` o `threshold.config` |

## Checklist

- [ ] Sistema actualizado y Suricata instalado
- [ ] Reglas descargadas con `suricata-update`
- [ ] `HOME_NET`, interfaz y `community-id` configurados
- [ ] `local.rules` creado e incluido en `rule-files`
- [ ] `suricata -T` sin errores
- [ ] Servicio habilitado con `systemctl enable --now suricata`
- [ ] Alerta de `testmynids.org` visible en `fast.log`
- [ ] Actualización de reglas programada

---
📎 Guías: ![[Suricata-Guia.pdf]]

🔗 Relacionado: [[Snort]] · [[iptables]] · [[pfSense en VirtualBox]] · [[Portal cautivo en pfSense]] · [[Splunk Enterprise]] · [[Nmap]] · [[Plan de Respuesta a Incidentes (PRI)]]
