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
---

# Spanning Tree Protocol (STP)

STP previene **bucles de red** en topologías de switches con caminos redundantes, bloqueando los enlaces sobrantes para que solo uno esté activo entre dos dispositivos.

## ¿Por qué importa?
1. **Previene bucles** (que congestionan la red y causan pérdida de datos).
2. **Aumenta la disponibilidad** al mantener la topología estable.
3. **Redundancia**: si el camino activo falla, STP activa uno alternativo.

## Configuración

**1. Habilitar el modo (Rapid PVST+):**

```cisco
Switch(config)# spanning-tree mode rapid-pvst
```

**2. Prioridad del bridge (raíz)** — dos métodos:

*Método 1 — root primario / secundario:*

```cisco
spanning-tree vlan 10 root primary     ! switch principal (prioridad más baja)
spanning-tree vlan 10 root secondary   ! respaldo
```

*Método 2 — prioridad manual (menor = mayor prioridad):*

```cisco
Switch(config)# spanning-tree vlan 10 priority 4096
```

**3. PortFast** (puertos hacia dispositivos finales: PCs, impresoras):

```cisco
Switch(config)# interface range fa0/1 - 24
Switch(config-if-range)# spanning-tree portfast
```

**4. BPDU Guard** (protege puertos PortFast: si reciben un BPDU se desactivan):

```cisco
Switch(config-if-range)# spanning-tree bpduguard enable
```

**5. Verificar:**

```cisco
Switch# show spanning-tree
```

## Resumen de comandos

| Comando | Descripción |
|---|---|
| `spanning-tree mode rapid-pvst` | Habilita el modo STP |
| `spanning-tree vlan [n] priority [valor]` | Prioridad del switch en esa VLAN |
| `interface range fa0/1 - 24` | Selecciona rango de puertos |
| `spanning-tree portfast` | Habilita PortFast en el puerto |
| `spanning-tree bpduguard enable` | Habilita BPDU Guard |
| `show spanning-tree` | Estado actual de STP |

> [!warning] Solo en puertos de acceso
> PortFast/BPDU Guard van en puertos finales, **nunca en trunks**. La protección STP (Root Guard, Loop Guard) se trata en [[Seguridad en la Capa 2]].

---
📎 Guía completa: ![[STP-Spanning-Tree.pdf]]

🔗 Relacionado: [[EtherChannel con LACP]] · [[VLAN y enrutamiento Inter-VLAN]] · [[Seguridad en la Capa 2]]
