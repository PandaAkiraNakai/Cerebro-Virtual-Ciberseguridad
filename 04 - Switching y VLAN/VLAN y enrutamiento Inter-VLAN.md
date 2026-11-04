---
tags:
  - cisco
  - switching
  - vlan
  - enrutamiento
aliases:
  - VLAN
  - Inter-VLAN
  - Router-on-a-stick
  - dot1Q
  - VLAN nativa
---

# VLAN y enrutamiento Inter-VLAN

Una **VLAN** (*Virtual LAN*) es una subred lógica dentro de la misma infraestructura de switches. Agrupa equipos por función sin importar el puerto físico y parte un dominio de broadcast grande en varios pequeños. Equipos de VLAN distintas solo se hablan a través de un dispositivo de **capa 3** (router o switch multicapa).

> [!tip] Regla de oro
> 1 VLAN = 1 dominio de broadcast = 1 subred IP.

| Característica | Red plana | Red con VLAN |
|---|---|---|
| Dominio de broadcast | Uno para toda la red | Uno por VLAN |
| Seguridad | Baja: todo el tráfico es visible | Aislamiento por función |
| Escalabilidad | Se degrada con muchos equipos | Fácil agregar subredes |

## Tipos de VLAN

| Tipo | Uso |
|---|---|
| Datos | Tráfico de usuarios finales |
| Voz | Telefonía IP; requiere QoS (ver [[QoS en router y switch con FreePBX]]) |
| Gestión | Administración remota de switches, routers y AP (SSH, SNMP) |
| Nativa | Tráfico **sin etiqueta** en un troncal; cámbiala por seguridad |
| Predeterminada | VLAN 1 en Cisco; no se puede borrar ni renombrar |

| Rango | ID | Dónde se guarda |
|---|---|---|
| Normal | 1-1005 (1002-1005 reservadas) | `vlan.dat` |
| Extendido | 1006-4094 | `running-config`; usado por ISP y data centers |

## Configuración del switch

```cisco
! Crear VLAN
vlan 10
 name Ventas
vlan 20
 name Voz_IP

! Puerto de acceso (sin etiqueta)
interface range fastethernet0/1 - 10
 switchport mode access
 switchport access vlan 10

! Troncal 802.1Q (con etiqueta) hacia otro switch o el router
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 switchport trunk native vlan 999         ! nativa sin uso, distinta de la 1
 switchport nonegotiate                   ! apaga DTP

! IP de administración (SVI)
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
ip default-gateway 192.168.99.1
```

## Router-on-a-stick

El router enruta entre VLAN con **subinterfaces**, una por VLAN, sobre un solo enlace troncal.

```cisco
interface GigabitEthernet0/0/0
 no shutdown
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

La IP de cada subinterfaz es el gateway de su VLAN.

## Verificación

```cisco
show vlan brief            ! VLAN y puertos asignados
show interfaces trunk      ! troncales, VLAN permitidas y nativa
show interfaces status     ! estado de cada puerto
```

> [!warning] Seguridad de VLAN
> Usa una **VLAN nativa** distinta de la 1 y sin puertos, y desactiva DTP para evitar *VLAN hopping*. Ver [[Seguridad en la Capa 2]].

---
📎 Guía completa: ![[VLAN-Inter-VLAN.pdf]]

🔗 Relacionado: [[Spanning Tree Protocol (STP)]] · [[EtherChannel con LACP]] · [[Gestión de VLAN con VTP y MVRP]] · [[FHRP - HSRP, VRRP y GLBP]] · [[Seguridad en la Capa 2]] · [[QoS en router y switch con FreePBX]]
