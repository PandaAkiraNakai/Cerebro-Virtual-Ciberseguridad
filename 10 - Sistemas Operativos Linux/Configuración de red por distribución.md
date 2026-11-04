---
tags:
  - linux
  - redes
  - netplan
  - networkmanager
aliases:
  - netplan
  - nmcli
  - IP estática en Linux
  - /etc/network/interfaces
---

# Configuración de red por distribución

Cada familia de Linux guarda la configuración de red en un lugar distinto. Los comandos `ip` sirven en todas, pero lo que hacen **no persiste** tras reiniciar. Los diagnósticos generales están en [[02 - Red y conectividad]].

| Distribución | Herramienta por defecto | Archivo |
|---|---|---|
| Ubuntu (servidor) | netplan | `/etc/netplan/*.yaml` |
| Ubuntu (escritorio), Fedora, RHEL 9 | NetworkManager | `nmcli`, `nmtui` |
| Debian | ifupdown | `/etc/network/interfaces` |
| Alpine | ifupdown (BusyBox) | `/etc/network/interfaces` |
| TinyCore | Scripts de arranque | `/opt/bootlocal.sh` |

## Consultar

| Comando | Muestra |
|---|---|
| `ip -br a` | Interfaces e IP en formato breve |
| `ip route` | Rutas y gateway |
| `ip link` | Estado up/down, MTU y MAC |
| `hostname -I` | Solo las IP asignadas |
| `resolvectl status` o `cat /etc/resolv.conf` | Servidores DNS |

## Ubuntu con netplan

```yaml
# /etc/netplan/01-netcfg.yaml — DHCP
network:
  version: 2
  ethernets:
    ens2:
      dhcp4: true
```

```yaml
# IP estática
network:
  version: 2
  ethernets:
    ens2:
      dhcp4: false
      addresses:
        - 192.168.0.10/24
        - 192.168.0.11/24          # varias IP en la misma interfaz
      routes:
        - to: default
          via: 192.168.0.1
        - to: 10.0.0.0/8           # ruta estática adicional
          via: 192.168.0.254
      nameservers:
        addresses: [1.1.1.1, 9.9.9.9]
```

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml   # netplan advierte si el archivo es legible por otros
sudo netplan generate                        # valida la sintaxis
sudo netplan try                             # aplica y revierte solo si no confirmas en 120 s
sudo netplan apply
```

> [!warning] `gateway4` está obsoleto
> Las guías antiguas usan `gateway4:`. En netplan actual se usa `routes:` con `to: default`.

> [!tip] Usa `netplan try` por SSH
> Si la nueva configuración te deja sin conexión, se revierte sola.

Bridge para laboratorios (por ejemplo, para colgar VMs a la red física):

```yaml
network:
  version: 2
  ethernets:
    ens2: { dhcp4: false }
    ens3: { dhcp4: false }
  bridges:
    br0:
      interfaces: [ens2, ens3]
      dhcp4: true
```

Verifica con `ip link show br0` o `bridge link`.

## Debian y Alpine con ifupdown

```text
# /etc/network/interfaces — DHCP
auto eth0
iface eth0 inet dhcp

# Estática
auto eth0
iface eth0 inet static
    address 192.168.0.10/24
    gateway 192.168.0.1
    dns-nameservers 1.1.1.1 9.9.9.9
```

`dns-nameservers` requiere el paquete `resolvconf`; sin él, edita `/etc/resolv.conf`.

```bash
sudo ifdown eth0 && sudo ifup eth0       # Debian
sudo systemctl restart networking        # Debian
sudo rc-service networking restart       # Alpine
```

## NetworkManager (nmcli)

```bash
# DHCP
sudo nmcli connection add type ethernet ifname ens2 con-name LAN ipv4.method auto
# Estática
sudo nmcli connection add type ethernet ifname ens2 con-name LAN \
  ipv4.method manual ipv4.addresses 192.168.0.10/24 \
  ipv4.gateway 192.168.0.1 ipv4.dns "1.1.1.1 9.9.9.9"
sudo nmcli connection up LAN

nmcli connection show
nmcli device status
sudo nmtui                               # interfaz de texto
```

En RHEL 9 y Fedora los perfiles se guardan en `/etc/NetworkManager/system-connections/`; los antiguos `ifcfg-eth0` de `/etc/sysconfig/network-scripts/` están obsoletos.

| Aspecto | nmcli | netplan |
|---|---|---|
| Dónde se usa | Fedora, RHEL, escritorios Ubuntu | Servidores Ubuntu |
| Persistencia | Automática | Archivo YAML |
| Mejor para | Equipos con Wi-Fi o redes cambiantes | Servidores e infraestructura como código |

## TinyCore

Usa `ifconfig` (no trae `ip`). Para que la configuración sobreviva, agrégala a `/opt/bootlocal.sh`:

```bash
ifconfig eth0 192.168.0.10 netmask 255.255.255.0 up
route add default gw 192.168.0.1
echo "nameserver 1.1.1.1" > /etc/resolv.conf
```

## Cambios temporales

```bash
sudo ip link set ens2 down && sudo ip link set ens2 up
sudo ip addr add 192.168.0.10/24 dev ens2
sudo ip addr del 192.168.0.10/24 dev ens2
sudo ip route add 10.0.0.0/8 via 192.168.0.1
sudo dhclient -r ens2 && sudo dhclient ens2      # renovar DHCP (en sistemas con dhclient)
```

## Diagnóstico

| Comando | Muestra |
|---|---|
| `ethtool ens2` | Velocidad, dúplex y enlace |
| `ip neigh` | Tabla ARP |
| `ss -tulpn` | Puertos a la escucha |
| `traceroute 1.1.1.1` | Saltos hasta el destino |
| `dig ejemplo.com` | Consulta DNS detallada |
| `journalctl -u systemd-networkd -u NetworkManager -n 50` | Logs del gestor de red |

---
🔗 Relacionado: [[02 - Red y conectividad]] · [[12 - Troubleshooting y diagnóstico]] · [[SSH en Linux]] · [[11 - Virtualización y contenedores]]
