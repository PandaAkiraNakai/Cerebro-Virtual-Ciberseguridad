---
tags:
  - cisco
  - switching
  - vlan
  - enrutamiento
aliases:
  - Inter-VLAN
  - Router-on-a-stick
  - dot1Q
---

# VLAN y enrutamiento Inter-VLAN

Las **VLANs** segmentan una red en subredes más pequeñas para mejorar rendimiento y seguridad. El **enrutamiento Inter-VLAN** (router-on-a-stick) permite la comunicación entre ellas usando **subinterfaces** y encapsulación **802.1Q**.

## Objetivos
- Crear y asignar una VLAN en el switch.
- Configurar subinterfaz en el router para enrutar entre VLANs.
- Configurar un trunk en el switch.
- Verificar y probar la comunicación.

## Configuración del router (subinterfaz)

```cisco
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
```

## Configuración del switch

**Crear y asignar la VLAN 10:**

```cisco
Switch(config)# vlan 10
Switch(config-vlan)# name VLAN10
Switch(config-vlan)# exit

Switch(config)# interface range fastethernet0/1 - 10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit
```

**Configurar el trunk (hacia el router):**

```cisco
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10
Switch(config-if)# exit
```

## Verificación

```cisco
Switch# show vlan brief
Switch# show interface trunk
```

> [!warning] Seguridad de VLAN
> Configura la **VLAN nativa** distinta de la VLAN 1 y desactiva DTP para evitar *VLAN hopping*. Ver [[Seguridad en la Capa 2]].

---
📎 Guía completa: ![[VLAN-Inter-VLAN.pdf]]

🔗 Relacionado: [[Spanning Tree Protocol (STP)]] · [[EtherChannel con LACP]] · [[Seguridad en la Capa 2]] · [[QoS en router y switch con FreePBX]]
