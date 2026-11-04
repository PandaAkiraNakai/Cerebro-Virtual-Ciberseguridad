---
tags:
  - cisco
  - switching
  - stp
  - redundancia
aliases:
  - STP
  - Spanning Tree
  - RPVST
  - Rapid PVST+
  - RSTP
  - MSTP
---

# Spanning Tree Protocol (STP)

La redundancia física en capa 2 crea caminos cerrados, y Ethernet **no tiene TTL**: una trama puede dar vueltas para siempre. Spanning Tree bloquea lógicamente los enlaces sobrantes y los deja de respaldo; si el camino activo cae, activa el alternativo.

```text
      ┌─────┐
      │ SW1 │
      └┬───┬┘
   ┌───┘   └───┐
┌──┴──┐     ┌──┴──┐
│ SW2 ├─────┤ SW3 │     broadcast: SW1 → SW2 → SW3 → SW1 → …
└─────┘     └─────┘
```

| Problema de un bucle | Efecto |
|---|---|
| Tormenta de broadcast | CPU y enlaces al 100 % |
| Inestabilidad de la tabla MAC | La MAC salta entre puertos |
| Tramas duplicadas | Los hosts reciben varias copias |

## Versiones

| Versión | Estándar | Instancias | Convergencia |
|---|---|---|---|
| STP | IEEE 802.1D | 1 para todas las VLAN | 30-50 s |
| PVST+ | Cisco | 1 por VLAN | 30-50 s |
| RSTP | IEEE 802.1w | 1 para todas | < 1 s a pocos s |
| Rapid PVST+ | Cisco | 1 por VLAN | < 1 s a pocos s |
| MSTP | IEEE 802.1s | VLAN agrupadas en instancias | < 1 s a pocos s |

> [!warning] STP clásico no sirve en producción
> 30 a 50 segundos de corte interrumpen VoIP, sesiones y aplicaciones. Usa **Rapid PVST+** o **MSTP**.

## Cómo elige el árbol (802.1D)

Los switches intercambian **BPDU** cada 2 segundos.

```text
Bridge ID = prioridad (4 bits) + Extended System ID (12 bits = VLAN) + MAC (48 bits)
VLAN 10 con prioridad por defecto → 32768 + 10 = 32778 . 0011.2233.4455
```

La prioridad por defecto es **32768** y solo admite **múltiplos de 4096**.

Algoritmo STA:

1. **Root bridge:** el de menor Bridge ID.
2. **Root port:** en cada switch no raíz, el de menor costo hacia el root.
3. **Designated port:** uno por segmento.
4. El resto queda **bloqueado** (non-designated).

Desempate: menor costo al root → menor Bridge ID del emisor → menor Port ID del emisor.

| Velocidad | Costo (802.1D revisado) |
|---|---|
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

### Estados de puerto (802.1D)

| Estado | Recibe BPDU | Aprende MAC | Reenvía | Duración |
|---|---|---|---|---|
| Blocking | Sí | No | No | 20 s (Max Age) |
| Listening | Sí | No | No | 15 s |
| Learning | Sí | Sí | No | 15 s |
| Forwarding | Sí | Sí | Sí | — |
| Disabled | No | No | No | — |

## RSTP (802.1w)

Mantiene la lógica de elección, pero converge en milisegundos con el handshake **Proposal / Agreement** entre vecinos.

| Característica | STP 802.1D | RSTP 802.1w |
|---|---|---|
| Detección de falla | Max Age 20 s | 3 Hellos perdidos (6 s) |
| Quién genera BPDU | Solo el root | Cada switch |
| Transición | Por temporizadores | Proposal / Agreement |
| Puertos de respaldo | No | Alternate / Backup |

Estados: Disabled, Blocking y Listening se funden en **Discarding**; quedan Learning y Forwarding.

| Rol | Función |
|---|---|
| Root | Mejor camino al root bridge |
| Designated | Reenvía en el segmento |
| **Alternate** | Camino alternativo al root; pasa a Forwarding de inmediato si cae el root port |
| **Backup** | Respaldo de un designated en el mismo segmento |
| Disabled | Apagado administrativamente |

| Tipo de enlace | Descripción | Convergencia rápida |
|---|---|---|
| Edge | Hacia hosts (PortFast) | Sí |
| Point-to-point | Switch a switch, full-duplex | Sí |
| Shared | Half-duplex (hub) | No |

## Configuración con Rapid PVST+

```cisco
spanning-tree mode rapid-pvst

! Root: método automático…
spanning-tree vlan 10,20 root primary      ! en el switch principal
spanning-tree vlan 10,20 root secondary    ! en el respaldo
! …o prioridad manual (menor = más prioridad, múltiplos de 4096)
spanning-tree vlan 10 priority 4096

! Puertos hacia hosts (edge)
interface range fa0/1 - 24
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable            ! apaga el puerto si llega una BPDU

! Enlaces entre switches
interface GigabitEthernet0/1
 spanning-tree link-type point-to-point
```

> [!warning] PortFast solo hacia hosts
> PortFast y BPDU Guard van en puertos de acceso, **nunca en enlaces entre switches**. Root Guard y Loop Guard se tratan en [[Seguridad en la Capa 2]].

## MSTP (802.1s)

PVST+ crea un árbol por VLAN: con 500 VLAN son 500 árboles. MSTP agrupa las VLAN en **pocas instancias** y ahorra CPU y memoria.

```text
VLAN 10, 20, 30 → instancia 1 → root DSW1
VLAN 40, 50, 60 → instancia 2 → root DSW2
resto           → instancia 0 (IST)
```

Dos switches están en la misma **región MST** solo si coinciden **nombre**, **revisión** y **mapeo VLAN → instancia**. La IST (instancia 0) siempre existe y habla con los switches fuera de la región. Con dos instancias y roots cruzados, **ambos enlaces de subida trabajan**.

```cisco
spanning-tree mode mst
spanning-tree mst configuration
 name EMPRESA
 revision 1
 instance 1 vlan 10,20,30
 instance 2 vlan 40,50,60
 exit
spanning-tree mst 1 priority 4096     ! DSW1
spanning-tree mst 2 priority 8192
! En DSW2 se invierten: mst 1 priority 8192 · mst 2 priority 4096
```

## Comparación

| Característica | STP | RSTP | PVST+ | Rapid PVST+ | MSTP |
|---|---|---|---|---|---|
| Estándar abierto | Sí | Sí | No | No | Sí |
| Convergencia rápida | No | Sí | No | Sí | Sí |
| Balanceo por VLAN | No | No | Sí | Sí | Sí |
| Uso de recursos | Bajo | Bajo | Alto | Alto | Bajo |
| Uso habitual | Legado | Básico | Legado | Redes Cisco | Multivendor |

## Verificación

```cisco
show spanning-tree
show spanning-tree vlan 10
show spanning-tree summary          ! modo activo
show spanning-tree root
show spanning-tree mst configuration
show spanning-tree interface GigabitEthernet0/1 detail
```

## Errores frecuentes

| Error | Efecto o mensaje |
|---|---|
| No fijar el root | Gana la MAC más baja, a menudo el switch más viejo |
| Modos distintos entre switches | Convergencia lenta o impredecible (`show spanning-tree summary`) |
| PortFast entre switches | Bucles temporales |
| Región MST distinta (nombre, revisión o mapeo) | La región se parte |
| Prioridad no múltiplo de 4096 | `% Bridge Priority must be in increments of 4096` |

> [!tip] Alinea STP con FHRP
> El root bridge de cada VLAN debe coincidir con el gateway activo de [[FHRP - HSRP, VRRP y GLBP|HSRP o VRRP]].

## Checklist

- [ ] Elegir Rapid PVST+ o MSTP
- [ ] Root primario y secundario definidos
- [ ] PortFast y BPDU Guard en puertos de acceso
- [ ] Enlaces entre switches como point-to-point
- [ ] Verificar roles, estados y región MST
- [ ] Simular la caída de un enlace y medir la convergencia

---
📎 Guía completa: ![[STP-Spanning-Tree.pdf]]

🔗 Relacionado: [[EtherChannel con LACP]] · [[VLAN y enrutamiento Inter-VLAN]] · [[Gestión de VLAN con VTP y MVRP]] · [[FHRP - HSRP, VRRP y GLBP]] · [[Seguridad en la Capa 2]]
