---
tags:
  - cisco
  - qos
  - voip
aliases:
  - QoS solo router
  - QoS voz sin switch
---

# QoS Cisco para voz (sin switch administrable)

**QoS** prioriza tráfico sensible (voz VoIP) frente a datos. Sin un switch administrable, la configuración se aplica **directamente en el router**, clasificando por puertos UDP típicos de voz (RTP/SIP) a **nivel 3/4**.

## 1. Definir las clases de tráfico

```cisco
class-map match-any VOZ
 match protocol rtp audio
 match access-group name VOZ-UDP

class-map match-any SIGNAL
 match protocol sip
 match access-group name SIP-UDP
```

## 2. ACLs para identificar el tráfico

```cisco
ip access-list extended VOZ-UDP
 permit udp any any range 16384 32767

ip access-list extended SIP-UDP
 permit udp any any eq 5060
```

## 3. Política de priorización

```cisco
policy-map QOS-VOZ
 class VOZ
  priority 1500
 class SIGNAL
  bandwidth 256
 class class-default
  fair-queue
```

## 4. Aplicar en la interfaz LAN/WAN

```cisco
interface GigabitEthernet0/0
 description Enlace hacia Internet
 service-policy output QOS-VOZ
```

## Verificación

```cisco
show policy-map interface GigabitEthernet0/0
show class-map
show policy-map
```

> [!note] Nivel 2 vs nivel 3/4
> Sin switch administrable **no se pueden marcar tramas a nivel 2 (CoS)**; la clasificación se hace a nivel **3/4 (IP/UDP)**. Útil en entornos domésticos o redes pequeñas donde el router es el gateway. Para marcado CoS/DSCP completo ver [[QoS en router y switch con FreePBX]].

---
🔗 Relacionado: [[QoS en router y switch con FreePBX]] · [[Capacity Planning y QoS]] · [[ACL extendida]]
