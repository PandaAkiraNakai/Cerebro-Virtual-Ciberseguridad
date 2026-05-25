---
tags:
  - cisco
  - seguridad
  - capa-2
  - switching
aliases:
  - Seguridad L2
  - Seguridad Capa 2
---

# Seguridad en la Capa 2

La **Capa 2 (enlace de datos)** es fundamental: si se compromete, se afectan todas las capas superiores. Esta nota mapea los ataques L2 más comunes y sus mitigaciones.

> [!abstract] Mapa de defensas L2
> | Ataque | Mitigación | Nota detallada |
> |---|---|---|
> | MAC Flooding / Spoofing | [[Port Security]] | ✅ |
> | VLAN Hopping / doble etiquetado | Seguridad de VLANs | abajo |
> | DHCP Starvation / Spoofing | [[DHCP Snooping]] | ✅ |
> | ARP Spoofing / Poisoning | Dynamic ARP Inspection (DAI) | abajo |
> | Suplantación IP/MAC | IP Source Guard (IPSG) | abajo |
> | Manipulación de root bridge / bucles | Seguridad STP | abajo |

## 1. Port Security
Limita las MAC permitidas por puerto. Evita MAC Flooding y suplantación. → [[Port Security]]

```cisco
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum <1-8192>
Switch(config-if)# switchport port-security mac-address sticky
Switch(config-if)# switchport port-security violation {shutdown | restrict | protect}
```

## 2. Seguridad de VLANs
Evita VLAN hopping y doble etiquetado.

```cisco
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan <vlan-id>
Switch(config-if)# switchport trunk native vlan <vlan-id>   ! evitar VLAN 1
Switch(config-if)# switchport nonegotiate                   ! desactiva DTP
Switch(config-if)# switchport protected                     ! aísla puertos (PVLAN Edge)
Switch(config-if)# shutdown                                 ! deshabilita puertos no usados
```

## 3. DHCP Snooping
Solo permite respuestas de servidores DHCP legítimos. → [[DHCP Snooping]]

```cisco
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan <vlan-id|rango>
Switch(config-if)# ip dhcp snooping trust          ! puerto hacia el servidor
Switch(config-if)# ip dhcp snooping limit rate <rate>
```

## 4. Dynamic ARP Inspection (DAI)
Verifica IP-MAC en tramas ARP usando la base de DHCP Snooping. Previene ARP Spoofing/Poisoning.

```cisco
Switch(config)# ip arp inspection vlan <vlan-id|rango>
Switch(config-if)# ip arp inspection trust
Switch(config)# ip arp inspection validate {src-mac | dst-mac | ip}
Switch(config-if)# ip arp inspection limit rate <pps>
```

## 5. IP Source Guard (IPSG)
Bloquea tráfico IP/MAC no autorizado según los registros de DHCP Snooping.

```cisco
Switch(config-if)# ip verify source
Switch(config-if)# ip verify source port-security    ! valida IP + MAC
```

## 6. Seguridad STP
Protege el [[Spanning Tree Protocol (STP)|Spanning Tree]] contra manipulación de roles y bucles.

```cisco
Switch(config-if)# spanning-tree portfast
Switch(config-if)# spanning-tree bpduguard enable     ! apaga puerto que recibe BPDU
Switch(config-if)# spanning-tree guard root           ! evita root no autorizado
Switch(config-if)# spanning-tree guard loop           ! evita bucles por pérdida de BPDU
Switch(config-if)# spanning-tree bpdufilter enable
```

## 7. Endurecimiento global del switch

```cisco
Switch(config)# no cdp run                                  ! desactiva CDP
Switch(config)# no lldp run                                 ! desactiva LLDP
Switch(config)# banner motd $Unauthorized Access Prohibited$
Switch(config)# line console 0
Switch(config-line)# exec-timeout 5 0                       ! cierra sesiones inactivas
Switch(config)# logging buffered 16384
Switch(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Switch(config)# line vty 0 15
Switch(config-line)# access-class 10 in                     ! ACL para acceso remoto
```

## Comandos de verificación

```cisco
show port-security
show ip dhcp snooping            ;  show ip dhcp snooping binding
show ip arp inspection           ;  show ip arp inspection statistics
show ip verify source
show spanning-tree summary       ;  show spanning-tree inconsistentports
show interfaces status           ;  show interfaces switchport
show vlan brief
```

---
🔗 Relacionado: [[Port Security]] · [[DHCP Snooping]] · [[Spanning Tree Protocol (STP)]] · [[Protocolos de descubrimiento y sincronización (CDP, LLDP, NTP)]]
