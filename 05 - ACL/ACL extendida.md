---
tags:
  - cisco
  - acl
  - seguridad
aliases:
  - ACL extendida
---

# ACL Extendida

Filtran tráfico por **IP de origen**, **IP de destino**, **protocolo** (TCP, UDP, ICMP…) y **puerto**, permitiendo un control mucho más preciso.

- **Numeración:** 100–199 y 2000–2699.
- Se aplican **cerca del origen**.

## Ejemplos

**Permitir HTTP desde una red:**

```cisco
access-list 110 permit tcp 192.168.1.0 0.0.0.255 any eq 80
access-list 110 deny ip any any
```

**Permitir solo SSH:**

```cisco
access-list 120 permit tcp 192.168.10.0 0.0.0.255 any eq 22
access-list 120 deny ip any any
```

**Permitir SSH y Telnet:**

```cisco
access-list 122 permit tcp 192.168.10.0 0.0.0.255 any eq 22
access-list 122 permit tcp 192.168.10.0 0.0.0.255 any eq 23
access-list 122 deny ip any any
```

**Permitir HTTP y HTTPS:**

```cisco
access-list 123 permit tcp any any eq 80
access-list 123 permit tcp any any eq 443
access-list 123 deny ip any any
```

**Bloquear ICMP (ping):**

```cisco
access-list 124 deny icmp any any
access-list 124 permit ip any any
```

**Permitir acceso a un servidor específico (HTTP):**

```cisco
access-list 125 permit tcp any host 192.168.20.10 eq 80
access-list 125 deny ip any any
```

**Permitir comunicación entre dos redes:**

```cisco
access-list 126 permit ip 192.168.1.0 0.0.0.255 10.10.10.0 0.0.0.255
access-list 126 deny ip any any
```

## Aplicar la ACL en la interfaz

```cisco
interface GigabitEthernet0/0
 ip access-group 120 in
```

> [!tip] Puertos comunes: `22` SSH · `23` Telnet · `80` HTTP · `443` HTTPS · `53` DNS · `5060` SIP.

---
🔗 Relacionado: [[ACL - Conceptos]] · [[ACL estándar]] · [[QoS en router y switch con FreePBX]]
