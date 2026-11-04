---
tags:
  - firewall
  - pfsense
  - virtualbox
  - laboratorio
aliases:
  - pfSense CE
  - Instalar pfSense
  - pfSense VirtualBox
---

# pfSense en VirtualBox

Instalación de **pfSense Community Edition** como firewall y router virtual en Oracle VirtualBox, con dos interfaces: **WAN** hacia la red externa y **LAN** para una red interna aislada de laboratorio.

```text
   Internet
      │
 VirtualBox NAT
      │ em0 (WAN, DHCP)
  ┌───┴─────┐
  │ pfSense │
  └───┬─────┘
      │ em1 (LAN) 192.168.1.1/24
   Red interna "intnet" · DHCP .100-.150
      │
 clientes Linux / Windows / servidores
```

## Máquina virtual

| Parámetro | Valor |
|---|---|
| Tipo / versión | BSD / FreeBSD (64-bit) |
| vCPU | 2 |
| RAM | 4096 MB (ajustable al anfitrión) |
| Disco | VDI dinámico de 20 GB o más |
| Adaptador 1 | **NAT** → será `em0` (WAN) |
| Adaptador 2 | **Red interna** `intnet` → será `em1` (LAN) |

> [!warning] WAN y LAN en redes distintas
> Si ambas interfaces quedan en la misma red, pfSense no puede enrutar ni filtrar entre ellas.

Monta la ISO de pfSense CE en **Configuración → Almacenamiento** y arranca la VM.

## Instalador

1. **Copyright and Distribution Notice** → Accept.
2. **Welcome to pfSense!** → Install. Confirma la inicialización de red.
3. **WAN Interface Assignment** → `em0`, modo **DHCP (client)**, VLAN desactivada.
4. **LAN Interface Assignment** → `em1`, modo **STATIC** `192.168.1.1/24`, DHCP de `192.168.1.100` a `192.168.1.150`.
5. Verifica el mapeo: Adaptador 1 → `em0` → WAN · Adaptador 2 → `em1` → LAN.
6. **Active Subscription Validation** → **Install CE**.
7. **Installation Options** → **ZFS** con **GPT**. En **ZFS Configuration**, `stripe` (un solo disco, sin redundancia).
8. **Disk Selection** → marca `ada0` con Espacio → OK. Confirma **Last Chance!** con Yes: borra el disco.
9. **Software Version to Install** → la versión CE disponible. Espera la descarga e instalación de paquetes sin apagar la VM.
10. Reinicia y **desmonta la ISO** para que no vuelva a arrancar el instalador.

## Primer acceso

En la consola debe verse `WAN → em0` y `LAN → em1` con `192.168.1.1/24`. Desde un cliente conectado a `intnet` que reciba IP por DHCP, abre `https://192.168.1.1`. Las credenciales iniciales del webGUI son `admin` / `pfsense`; el asistente pide cambiarlas.

## Verificación

- [ ] Arranca desde el disco y no desde la ISO
- [ ] `em0` es WAN con DHCP; `em1` es LAN con `192.168.1.1/24`
- [ ] DHCP de la LAN activo en `.100-.150`
- [ ] GPT + ZFS stripe
- [ ] Un cliente de la LAN navega a través de pfSense

## Para qué sirve después

Con la red interna aislada puedes colgar clientes, servidores y máquinas vulnerables sin tocar la red física, y practicar reglas de firewall, NAT, servicios de red, monitoreo y pruebas de seguridad controladas. Siguiente paso: [[Portal cautivo en pfSense]].

---
🔗 Relacionado: [[Portal cautivo en pfSense]] · [[FTDv en Cisco CML]] · [[iptables]] · [[Suricata]] · [[Laboratorio Kali Linux en Windows]]
