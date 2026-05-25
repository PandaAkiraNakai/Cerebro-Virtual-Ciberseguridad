---
tags:
  - cisco
  - qos
  - voip
  - freepbx
  - vlan
aliases:
  - QoS FreePBX
  - QoS router y switch
---

# QoS en router y switch (con FreePBX)

QoS prioriza tráfico sensible al retardo (voz/video). En redes con **FreePBX** u otra telefonía IP, evita **retardos, jitter y pérdida de paquetes** en las llamadas.

Cisco implementa QoS en cuatro procesos: **clasificación → marcado → priorización → control**. Aquí se combina QoS con una **VLAN exclusiva de voz**.

## Router Cisco

**1. ACL extendida para tráfico de voz (SIP/RTP):**

```cisco
Router(config)# access-list 100 permit udp any any eq 5060
Router(config)# access-list 100 permit udp any any range 5061 5062
Router(config)# access-list 100 permit udp any any range 10000 20000
```

> Ajusta el rango RTP según FreePBX: *Settings → Asterisk SIP Settings → RTP Port Range* (por defecto 10000–20000).

**2. Class-map:**

```cisco
Router(config)# class-map match-any VOZ
Router(config-cmap)# match access-group 100
```

**3. Policy-map (LLQ con 70 %):**

```cisco
Router(config)# policy-map QOS-VOZ
Router(config-pmap)# class VOZ
Router(config-pmap-c)# priority percent 70
Router(config-pmap)# class class-default
Router(config-pmap-c)# fair-queue
```

**4. Aplicar en la interfaz de salida** (recomendado en WAN y LAN):

```cisco
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# service-policy output QOS-VOZ
Router(config)# interface GigabitEthernet0/0/1
Router(config-if)# service-policy output QOS-VOZ
```

**Verificar:** `show access-list 100` · `show policy-map interface ...` · `show class-map`

## Switch Cisco

**1. VLAN de voz:**

```cisco
Switch(config)# vlan 20
Switch(config-vlan)# name VOZ
```

**2. Activar QoS globalmente:**

```cisco
Switch(config)# mls qos
```

**3. Puerto del teléfono IP / FreePBX:**

```cisco
Switch(config)# interface FastEthernet0/10
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10      ! datos (untagged)
Switch(config-if)# switchport voice vlan 20        ! voz (tagged 802.1Q)
Switch(config-if)# mls qos trust cos               ! confía en el CoS del teléfono
Switch(config-if)# spanning-tree portfast
```

> El puerto transporta **dos VLANs**: datos (untagged) y voz (tagged) **sin ser trunk**.

**4. Mapear CoS → DSCP:**

```cisco
Switch(config)# mls qos map cos-dscp 0 8 16 24 32 46 48 56
```

| CoS | DSCP | Uso |
|---|---|---|
| 0 | 0 | Best Effort |
| 5 | **46 (EF)** | **Voz** (RFC 3246 — prioridad absoluta) |
| 6 | 48 | Control |
| 7 | 56 | — |

**Verificar:** `show mls qos` · `show mls qos interface Fa0/10` · `show mls qos maps cos-dscp`

## Solución de problemas
- **Packets no suben en clase VOZ** → verifica la interfaz, el rango RTP y `show access-list 100`.
- **Drops en clase VOZ** → sube el porcentaje de prioridad (ej. 80 %) o revisa el ancho de banda.
- **El switch no marca** → confirma `mls qos` global y `mls qos trust cos` en el puerto.

> [!success] Resultado
> ACLs que clasifican SIP/RTP, VLAN de voz aislada, QoS en router y switch, y marcado CoS/DSCP coherente → **telefonía IP con calidad estable**.

---
📄 Codecs y dimensionamiento: [[Capacity Planning y QoS]]

🔗 Relacionado: [[QoS para voz sin switch administrable]] · [[VLAN y enrutamiento Inter-VLAN]] · [[ACL extendida]]
