---
tags:
  - reconocimiento
  - red
  - nmap
  - masscan
  - pentesting
  - recon-activo
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/network-discovery/"
licencia: MIT
---

# Reconocimiento Activo de Red

> [!warning] Uso ético y autorizado
> Genera tráfico hacia el objetivo. Aplicar solo con autorización. Ver [[📜 Fuentes y Licencias]].

---

## Nmap — Referencia de escaneos

### Escaneo básico (CTF / lab)
```bash
nmap -sV -sC -oA ~/nmap-initial <IP>
# -sV  detecta versión de servicios
# -sC  scripts por defecto (equivale a --script=default)
# -oA  guarda en los tres formatos (.nmap, .xml, .gnmap)

# Segunda pasada con todos los puertos
nmap -p- -sV -sC -oA ~/nmap-full <IP>
```

### Escaneo completo (producción / red real)
```bash
sudo nmap -sSV -p- -T4 -oA nmap-completo <IP>
# -sS  SYN scan (stealth)
# -p-  65535 puertos
# -T4  timing agresivo
```

### Escaneo agresivo (OS + scripts + traceroute)
```bash
nmap -A -T4 <IP>
```

### Ping sweep (descubrir hosts activos)
```bash
nmap -sn -n --disable-arp-ping 192.168.1.0/24 | grep -v "host down"
# -sn  sin escaneo de puertos
# -n   sin resolución DNS
```

### DHCP Discovery
```bash
sudo nmap --script broadcast-dhcp-discover
```

### Scripts útiles
```bash
# Enumerar usuarios SMB
nmap --script smb-enum-users.nse -p 445 <IP>

# Enumerar recursos web
nmap --script 'http-enum' -v <target> -p80

# Detectar vulnerabilidades con searchsploit
nmap -p- -sV -oX scan.xml <IP>
searchsploit --nmap scan.xml

# Generar reporte HTML
nmap -sV <IP> -oX scan.xml && xsltproc scan.xml -o "$(date +%m%d%y)_report.html"

# Listar todos los scripts disponibles
ls /usr/share/nmap/scripts/
```

---

## Masscan — Escaneo masivo de puertos

```bash
# Escaneo completo de una IP
masscan -e tun0 -p1-65535,U:1-65535 <IP> --rate 1000

# Escaneo de lista de IPs
masscan -iL ips.txt --rate 10000 -p1-65535 --only-open -oL masscan.out

# Descubrir máquinas activas en red
sudo masscan --rate 500 --interface eth0 --router-ip $GATEWAY --top-ports 100 192.168.1.0/24 -oL hosts.lst

# Workflow: masscan → nmap
TCP_PORTS=$(cat masscan.out | grep open | grep tcp | cut -d " " -f3 | tr '\n' ',' | head -c -1)
sudo nmap -sT -sC -sV -Pn -n -T4 -p$TCP_PORTS -oA nmap_tcp <IP>
```

---

## ARP Scan (LAN)

```bash
# Ver vecinos ARP actuales
ip neigh

# ARP scan con nmap (requiere root)
sudo nmap -sn -n 192.168.1.0/24

# ARP scan con arp-scan
sudo arp-scan -l

# Netdiscover (pasivo/activo)
netdiscover -i eth0 -r 192.168.1.0/24
```

---

## Ping Sweep (sin herramientas extra)

```bash
# Bash: sweep de toda la /24
for i in $(seq 1 255); do
  ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1 && echo "192.168.1.$i UP"
done

# Bash: sweep + puertos comunes por host
for i in $(seq 1 255); do
  ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1 || continue
  echo "192.168.1.$i UP:"
  for j in 21 22 80 139 443 445 3306 3389 8080 8443; do
    nc -z -w 1 192.168.1.$i $j > /dev/null 2>&1 && echo "  puerto $j abierto"
  done
done
```

---

## Protocolos de descubrimiento

### NBT-NS / NetBIOS
```bash
nbtscan -r 192.168.1.0/24        # escaneo de red
nmblookup -A <IP>                  # nombre de un host específico
```

### MDNS (Zeroconf / Bonjour)
```bash
mdns-scan
```

### LDAP (Directorio Activo)
```bash
# Null bind (sin credenciales)
ldapsearch -x -h <IP> -s base

# Buscar servicios AD por DNS
nslookup -type=srv _ldap._tcp.dc._msdcs.<dominio>
nslookup -type=srv _kerberos._tcp.<dominio>
```

### DNS Zone Transfer
```bash
host -t ns dominio.local
dig axfr dominio.local @<IP-del-NS>
```

---

## Reconnoitre (automatiza Nmap + nbtscan)

```bash
python2.7 reconnoitre.py -t 192.168.1.2-252 -o ./results/ \
  --pingsweep --hostnames --services --quick
```

---

## PowerShell (Windows)

```powershell
# Ping
tnc 8.8.8.8

# Escaneo de puerto
tnc 8.8.8.8 -port 443
```

---

## Flujo de trabajo recomendado

```
1. netdiscover / arp-scan   → Hosts activos en LAN
2. nmap -sn                  → Confirmar hosts y SO aproximado
3. masscan -p1-65535         → Todos los puertos rápido
4. nmap -sCV sobre resultados → Servicios, versiones, scripts
5. searchsploit --nmap       → Vulnerabilidades conocidas
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Reconocimiento Pasivo (OSINT)]]
- [[Reconocimiento Web]]
- [[Nmap]]
- Fuente: [InternalAllTheThings - Network Discovery](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/network-discovery/) — MIT
