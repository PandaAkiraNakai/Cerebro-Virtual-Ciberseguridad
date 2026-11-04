---
tags:
  - cisco
  - switching
  - redundancia
  - fhrp
aliases:
  - FHRP
  - HSRP
  - VRRP
  - GLBP
  - Redundancia de gateway
---

# FHRP — HSRP, VRRP y GLBP

Los hosts tienen **un solo default gateway**. Si ese router cae, la red queda sin salida aunque exista otro router. Los **FHRP** (*First Hop Redundancy Protocols*) crean un **router virtual** con IP y MAC virtuales compartidas entre varios routers físicos. Si el activo cae, otro asume la IP y la MAC virtual y el host no nota el cambio.

```text
        INTERNET
        │      │
     ┌──┴─┐  ┌─┴──┐
     │ R1 │  │ R2 │
     │ACT │  │STB │
     └──┬─┘  └─┬──┘
   .2   └──┬───┘  .3
     IP virtual .1 + MAC virtual
           │
   PC: gateway 192.168.10.1
```

| Protocolo | Estándar | Roles | Balanceo |
|---|---|---|---|
| HSRP | Cisco | Active / Standby | Por grupo |
| VRRP | IETF (RFC 3768 v2 / RFC 5798 v3) | Master / Backup | Por grupo |
| GLBP | Cisco | AVG / AVF | Nativo |

## HSRP — Hot Standby Router Protocol

| Elemento | Descripción |
|---|---|
| Active | Reenvía el tráfico del gateway virtual |
| Standby | Vigila al Active y lo reemplaza si falla |
| Prioridad | 0-255, por defecto **100**; gana la mayor |
| Preempt | Recupera el rol Active al volver. **No viene activado** |
| Timers | Hello 3 s · Hold 10 s |

| Versión | MAC virtual | Grupos | Multicast |
|---|---|---|---|
| HSRPv1 | `0000.0C07.ACxx` | 0-255 | 224.0.0.2 |
| HSRPv2 | `0000.0C9F.Fxxx` | 0-4095 | 224.0.0.102 |

```cisco
! R1 (principal)
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 standby version 2
 standby 1 ip 192.168.10.1
 standby 1 priority 110
 standby 1 preempt

! R2 (respaldo)
interface GigabitEthernet0/0
 ip address 192.168.10.3 255.255.255.0
 standby version 2
 standby 1 ip 192.168.10.1
 standby 1 priority 100
 standby 1 preempt
```

### Tracking de la WAN

Si cae la WAN del Active, este sigue vivo en la LAN y mantiene el rol, mandando tráfico a un camino muerto. El **tracking** le baja la prioridad para que el otro tome el control.

```cisco
track 1 interface GigabitEthernet0/1 line-protocol
interface GigabitEthernet0/0
 standby 1 track 1 decrement 20     ! 110 − 20 = 90 < 100 → R2 pasa a Active
```

Por alcanzabilidad real con IP SLA:

```cisco
ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet0/1
 frequency 5
ip sla schedule 1 life forever start-time now
track 2 ip sla 1 reachability
interface GigabitEthernet0/0
 standby 1 track 2 decrement 20
```

> [!warning] Tracking sin preempt no sirve
> La prioridad baja, pero el otro router no asume el rol si no tiene `preempt`. Configúralo en **ambos**.

### Balanceo con varios grupos

Un grupo por VLAN, con el Active repartido:

```cisco
! R1: Active en VLAN 10, respaldo en VLAN 20
interface Vlan10
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
interface Vlan20
 standby 20 ip 192.168.20.1
 standby 20 priority 90
 standby 20 preempt
```

> [!tip] Alinea FHRP con STP
> El Active HSRP de cada VLAN debe ser también el **root bridge** de esa VLAN (ver [[Spanning Tree Protocol (STP)]]). Si no, el tráfico cruza de más entre los switches de distribución.

## VRRP — Virtual Router Redundancy Protocol

Estándar abierto, funcionalmente parecido a HSRP. Es la opción en entornos **multivendor** (Cisco con Fortinet, Juniper o Linux con keepalived).

| Característica | HSRP | VRRP |
|---|---|---|
| Estándar | Cisco | IETF |
| Roles | Active / Standby | Master / Backup |
| Prioridad por defecto | 100 | 100 |
| Preempt por defecto | No | **Sí** |
| IP virtual = IP real | No | Sí (owner, prioridad 255) |
| Hello | 3 s | 1 s |
| MAC virtual | `0000.0C9F.Fxxx` (v2) | `0000.5E00.01xx` |
| Multicast | 224.0.0.102 (v2) | 224.0.0.18 |

```cisco
! VRRPv2 — R1
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 vrrp 1 ip 192.168.10.1
 vrrp 1 priority 110
! VRRPv2 — R2
interface GigabitEthernet0/0
 ip address 192.168.10.3 255.255.255.0
 vrrp 1 ip 192.168.10.1

! VRRPv3 (IOS XE)
fhrp version vrrp v3
interface GigabitEthernet0/0
 vrrp 1 address-family ipv4
  address 192.168.10.1 primary
  priority 110
  track 1 decrement 20
```

## GLBP — Gateway Load Balancing Protocol

HSRP y VRRP dejan un router ocioso por grupo. GLBP hace que **todos reenvíen** con una sola IP virtual.

| Rol | Función |
|---|---|
| AVG (Active Virtual Gateway) | Responde los ARP y entrega a cada host una MAC virtual distinta |
| AVF (Active Virtual Forwarder) | Reenvía el tráfico de su MAC virtual (hasta 4 por grupo) |

| Método | Comportamiento |
|---|---|
| `round-robin` | Alterna MAC en cada respuesta ARP (por defecto) |
| `weighted` | Según el peso de cada AVF |
| `host-dependent` | Un host siempre recibe la misma MAC |

```cisco
! R1
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 glbp 1 ip 192.168.10.1
 glbp 1 priority 110
 glbp 1 preempt
 glbp 1 load-balancing round-robin
! R2
interface GigabitEthernet0/0
 ip address 192.168.10.3 255.255.255.0
 glbp 1 ip 192.168.10.1
 glbp 1 load-balancing round-robin
```

> [!note] Poco usado hoy
> GLBP es propietario, con soporte limitado en plataformas nuevas y balanceo poco predecible. Se prefiere HSRP o VRRP con varios grupos, o diseños de capa 3 pura (ECMP, VXLAN/EVPN con anycast gateway).

## Comparación

| Característica | HSRP | VRRP | GLBP |
|---|---|---|---|
| Estándar abierto | No | Sí | No |
| Routers reenviando | 1 | 1 | Hasta 4 |
| Preempt por defecto | No | Sí | No |
| Balanceo nativo | No | No | Sí |
| Tracking | Sí | Sí | Sí (por peso) |
| Uso habitual | Redes Cisco | Multivendor | Poco usado |

## Verificación

```cisco
show standby brief      ! HSRP
show vrrp brief         ! VRRP
show glbp brief         ! GLBP
show track
```

```text
Interface  Grp  Pri P State   Active  Standby       Virtual IP
Gi0/0      1    110 P Active  local   192.168.10.3  192.168.10.1
```

En el host, `arp -a` debe mostrar la **MAC virtual** asociada al gateway.

## Errores frecuentes

| Error | Efecto |
|---|---|
| Hosts con la IP real del router como gateway | No hay conmutación; usa la IP virtual |
| Número de grupo distinto entre routers | Cada uno se cree Active |
| HSRPv1 en un lado y v2 en el otro | No interoperan |
| Tracking sin preempt | La prioridad baja pero nadie toma el rol |
| Decremento insuficiente (110 − 5 = 105 > 100) | No hay cambio |
| Active FHRP distinto del root bridge | Tráfico subóptimo entre distribución |

## Checklist

- [ ] IP virtual y número de grupo
- [ ] Prioridad mayor en el principal y `preempt` en ambos
- [ ] Tracking de la WAN con decremento suficiente
- [ ] Hosts con la IP virtual como gateway
- [ ] `show standby brief`
- [ ] Apagar el Active con `ping -t` corriendo y comprobar continuidad
- [ ] Restaurar y verificar la preemption

---
🔗 Relacionado: [[Spanning Tree Protocol (STP)]] · [[EtherChannel con LACP]] · [[VLAN y enrutamiento Inter-VLAN]] · [[Gestión de VLAN con VTP y MVRP]]
