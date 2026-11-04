---
tags:
  - cisco
  - switching
  - etherchannel
  - lacp
  - redundancia
aliases:
  - EtherChannel
  - LACP
  - PAgP
  - Port-Channel
  - Link Aggregation
---

# EtherChannel con LACP

**EtherChannel** agrupa varios enlaces físicos Ethernet en un único **enlace lógico** (*Port-Channel*). Suma ancho de banda, da redundancia y STP lo ve como un solo puerto, así que no bloquea los enlaces sobrantes.

```text
Sin EtherChannel                 Con EtherChannel
SW1 ════ SW2                     SW1 ══Po1══ SW2
    ──✖──  STP bloquea 3            4 × 1 Gbps = 4 Gbps
    ──✖──                           STP ve un solo enlace
    ──✖──
```

Hasta **8 enlaces activos** por grupo; LACP admite otros 8 en standby.

| Tipo | Protocolo | Estándar | Modos |
|---|---|---|---|
| LACP | Dinámico | IEEE 802.3ad / 802.1AX | Active / Passive |
| PAgP | Dinámico | Cisco | Desirable / Auto |
| Estático | Ninguno | — | On |

## Requisitos de compatibilidad

Todos los puertos del grupo deben coincidir en velocidad y dúplex, modo (access o trunk), VLAN de acceso o VLAN permitidas, VLAN nativa y tipo de medio. Si no, el puerto queda **suspendido** (`s`).

## LACP

| Modo | Comportamiento |
|---|---|
| **Active** | Inicia la negociación enviando LACPDU |
| **Passive** | Responde, pero no inicia |

| SW1 \ SW2 | Active | Passive |
|---|---|---|
| Active | Canal | Canal |
| Passive | Canal | **No hay canal** |

```cisco
! SW1
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-group 1 mode active
interface Port-channel1
 switchport mode trunk

! SW2
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-group 1 mode passive
```

Como acceso a una VLAN:

```cisco
interface port-channel 1
 switchport mode access
 switchport access vlan 10
```

Prioridades opcionales (deciden qué puertos quedan activos si hay más de 8):

```cisco
lacp system-priority 100
interface GigabitEthernet0/1
 lacp port-priority 100
```

> [!info] El estándar universal
> LACP funciona entre Cisco, Juniper, Aruba, Huawei, servidores Linux (bonding modo 802.3ad), VMware e hipervisores. Es la opción por defecto en diseños actuales.

## PAgP y modo On

| PAgP | Comportamiento |
|---|---|
| Desirable | Inicia la negociación |
| Auto | Responde; Auto con Auto no forma canal |

```cisco
interface range GigabitEthernet0/3 - 4
 switchport mode trunk
 channel-group 2 mode desirable      ! en el otro extremo: auto o desirable
```

PAgP es propietario y está en desuso. El modo **On** (`channel-group 3 mode on`) agrupa sin negociar: solo funciona con `on` al otro lado, no detecta errores de cableado y puede crear bucles si un extremo no quedó agrupado. Evítalo.

## Balanceo de carga

EtherChannel no reparte trama por trama. Aplica un **hash** sobre campos del encabezado y asigna cada **flujo** a un enlace.

> [!warning] Un flujo nunca supera un enlace físico
> Una sola descarga entre dos hosts usa un único enlace, aunque el canal sume 4 Gbps.

| Método | Campos | Uso |
|---|---|---|
| `src-mac` / `dst-mac` / `src-dst-mac` | MAC | Tráfico de capa 2 |
| `src-ip` | IP origen | Muchos clientes |
| `dst-ip` | IP destino | Muchos servidores |
| `src-dst-ip` | IP origen y destino | Recomendado en general |
| `src-dst-port` | Puertos L4 | Pocos hosts con muchos flujos |

```cisco
port-channel load-balance src-dst-ip     ! global; no necesita coincidir en ambos extremos
```

## EtherChannel de capa 3

Entre switches multicapa o routers, el Port-Channel puede llevar IP:

```cisco
interface range GigabitEthernet0/1 - 2
 no switchport
 channel-group 10 mode active
interface Port-channel10
 no switchport
 ip address 10.0.0.1 255.255.255.252
```

## Comparación

| Característica | LACP | PAgP | On |
|---|---|---|---|
| Estándar | Sí | No | — |
| Negociación y detección de errores | Sí | Sí | No |
| Multivendor | Sí | No | Sí |
| Enlaces en standby | Sí | No | No |
| Estado | Estándar | En desuso | Evitar |

## Verificación

```cisco
show etherchannel summary
show etherchannel port-channel
show etherchannel load-balance
show lacp neighbor
show lacp sys-id
show pagp neighbor
```

```text
Flags: D - down  P - bundled  I - stand-alone  s - suspended
       S - Layer2  U - in use  R - Layer3
Group  Port-channel  Protocol  Ports
1      Po1(SU)       LACP      Gi0/1(P) Gi0/2(P)
```

## Errores frecuentes

| Error | Síntoma |
|---|---|
| Passive con Passive, o Auto con Auto | No se forma el canal |
| LACP en un extremo y PAgP en el otro | Port-Channel en `(SD)` |
| Parámetros distintos entre puertos | Puerto en `s` (suspended) |
| Cambios solo en la interfaz física | Hay que aplicarlos en `interface Port-channel` |
| Esperar que un flujo use todo el canal | El hash lo ata a un enlace |

## Checklist

- [ ] Puertos compatibles
- [ ] LACP en ambos extremos, al menos uno en Active
- [ ] Configuración en la interfaz Port-channel
- [ ] Método de balanceo
- [ ] `show etherchannel summary` con los puertos en `P`
- [ ] Desconectar un enlace y comprobar continuidad

---
📎 Guía completa: ![[EtherChannel-LACP.pdf]]

🔗 Relacionado: [[Spanning Tree Protocol (STP)]] · [[VLAN y enrutamiento Inter-VLAN]] · [[FHRP - HSRP, VRRP y GLBP]] · [[Gestión de VLAN con VTP y MVRP]]
