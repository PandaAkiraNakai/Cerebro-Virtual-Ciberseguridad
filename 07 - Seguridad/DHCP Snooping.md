---
tags:
  - cisco
  - seguridad
  - capa-2
  - dhcp
aliases:
  - DHCP Snooping
---

# DHCP Snooping

Función de seguridad en switches que protege contra **DHCP spoofing**. Bloquea servidores DHCP no autorizados (*rogue*) y asegura que los clientes obtengan IPs solo de servidores legítimos. Mantiene una tabla confiable de *bindings* **IP-MAC-Puerto**.

## Objetivos
- Bloquear servidores DHCP no autorizados.
- Prevenir envenenamiento DHCP.
- Generar tabla confiable de bindings IP-MAC-Puerto.
- Limitar solicitudes DHCP excesivas.

## Configuración

```cisco
configure terminal

! 1. Habilitar globalmente
ip dhcp snooping

! 2. Proteger VLANs
ip dhcp snooping vlan 10,20-30

! (opcional) desactivar inserción de Opción 82 si causa problemas
no ip dhcp snooping information option

! 3. Puertos de confianza (hacia servidor DHCP legítimo)
interface GigabitEthernet0/1
 ip dhcp snooping trust
 exit

! 4. Limitar tasa de mensajes DHCP (opcional)
interface range FastEthernet0/2-24
 ip dhcp snooping limit rate 10
 exit
```

## Verificación

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
show ip dhcp snooping statistics
```

## Notas importantes
- **Todos los puertos son UNTRUSTED por defecto.**
- Solo los puertos hacia servidores DHCP legítimos deben ser `trust`.
- Funciona en switches de **capa 2**, no en routers.
- Requiere que los clientes obtengan IP por DHCP.
- Puertos no confiables bloquean automáticamente respuestas DHCP maliciosas.

> [!info] Base para DAI e IPSG
> La tabla de bindings de DHCP Snooping es la base de **Dynamic ARP Inspection** e **IP Source Guard**. Ver [[Seguridad en la Capa 2]].

---
🔗 Relacionado: [[Servicio DHCP en Cisco]] · [[Seguridad en la Capa 2]] · [[Port Security]]
