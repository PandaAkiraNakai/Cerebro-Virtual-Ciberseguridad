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
  - Port-Channel
  - Link Aggregation
---

# EtherChannel con LACP

**EtherChannel** agrupa varios enlaces físicos Ethernet en un único **enlace lógico**, sumando su ancho de banda y ofreciendo redundancia.

## Conceptos
- **EtherChannel**: agrupa 2–8 interfaces físicas en un *Port-Channel*.
- **Protocolos de negociación:**
  - **PAgP** (Port Aggregation Protocol): propietario de Cisco.
  - **LACP** (Link Aggregation Control Protocol): estándar **IEEE 802.3ad**.

## Requisitos
- **Misma configuración**: igual velocidad y dúplex en todas las interfaces.
- **2 a 8 interfaces** por grupo.
- **Mismas VLANs** o modo trunk consistente.

## Modos de LACP

| Modo | Comportamiento |
|---|---|
| **Active** | Inicia activamente la negociación enviando paquetes LACP. |
| **Passive** | Solo responde; espera que el otro extremo inicie. |

**Combinaciones válidas:**

| Puerto A | Puerto B | Resultado |
|---|---|---|
| Active | Active | ✅ Canal creado |
| Active | Passive | ✅ Canal creado |
| Passive | Passive | ❌ No se crea canal |

> [!note] Al menos un extremo debe estar en modo **Active** para que se forme el EtherChannel.

## Configuración (modo Active)

```cisco
Switch(config)# interface range gigabitEthernet 0/1 - 2
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# exit
```

Configurar el **Port-Channel** resultante como trunk o access:

```cisco
! Como trunk
Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

! Como access (VLAN específica)
Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan [VLAN_ID]
Switch(config-if)# exit
```

---
📎 Guía completa: ![[EtherChannel-LACP.pdf]]

🔗 Relacionado: [[Spanning Tree Protocol (STP)]] · [[VLAN y enrutamiento Inter-VLAN]]
