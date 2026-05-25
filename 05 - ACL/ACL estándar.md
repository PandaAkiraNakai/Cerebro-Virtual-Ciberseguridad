---
tags:
  - cisco
  - acl
  - seguridad
aliases:
  - ACL estándar
---

# ACL Estándar

Filtran tráfico basándose **únicamente en la IP de origen**. No permiten especificar protocolos ni puertos.

- **Numeración:** 1–99 y 1300–1999.
- Más simples y ligeras que las [[ACL extendida|extendidas]].
- Se aplican **cerca del destino** para evitar bloqueos innecesarios.
- También se usan con NAT (definir direcciones a traducir).

## Ejemplos

**Bloquear una IP, permitir el resto:**

```cisco
access-list 10 deny host 192.168.1.50
access-list 10 permit any
```

**Permitir solo una IP:**

```cisco
access-list 10 permit host 192.168.1.10
access-list 10 deny any
```

**Permitir una red completa:**

```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any
```

**Permitir múltiples redes:**

```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 permit 192.168.2.0 0.0.0.255
access-list 10 deny any
```

**Rango de IPs con máscara wildcard** (192.168.1.1 → .20):

```cisco
access-list 10 permit 192.168.1.1 0.0.0.19
access-list 10 deny any
```

**Con NAT (PAT):**

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

## Aplicar la ACL en la interfaz

```cisco
interface GigabitEthernet0/1
 ip access-group 10 in
```

> [!note] `in` aplica en dirección **entrante**; usa `out` para saliente.

---
🔗 Relacionado: [[ACL - Conceptos]] · [[ACL extendida]]
