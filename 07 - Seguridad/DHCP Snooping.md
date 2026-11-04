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

## Amenazas que mitiga

| Ataque | Qué hace |
|---|---|
| **Rogue DHCP** | Un servidor falso responde antes que el legítimo y entrega su propia IP como gateway o DNS: el tráfico pasa por el atacante (MitM; ver [[ARP Spoofing y MitM]]) |
| **DHCP starvation** | Miles de solicitudes con MAC falsas agotan el pool y nadie nuevo obtiene IP (DoS) |

## Puertos trusted y untrusted

| Tipo | Dónde | Mensajes permitidos |
|---|---|---|
| Trusted | Hacia el servidor DHCP legítimo, el router o troncales a otros switches | Todos |
| Untrusted (por defecto) | Puertos de acceso de usuarios | Solo los del cliente: DISCOVER y REQUEST. Un OFFER, ACK o NAK entrante se descarta |

Por cada concesión legítima en un puerto untrusted, el switch anota en la *binding table* la MAC, la IP, el tiempo de concesión, la VLAN y el puerto.

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

Ejemplo de salida:

```text
SW-1# show ip dhcp snooping binding
MacAddress         IpAddress   Lease(sec)  Type           VLAN  Interface
00:50:79:66:68:00  10.10.10.2  86223       dhcp-snooping  10    GigabitEthernet0/1
Total number of bindings: 1
```

| Equipo | Comando | Muestra |
|---|---|---|
| Router (servidor DHCP) | `show ip dhcp binding` | Concesiones entregadas |
| Switch | `show ip dhcp snooping binding` | Tabla de validación de capa 2 |

## Opción 82 y routers Cisco

Por defecto el switch agrega la **Opción 82** (*relay agent information*) a las solicitudes que entran por puertos untrusted. Un router Cisco que actúa como servidor DHCP descarta esas solicitudes si no está configurado para aceptarlas, y los clientes se quedan sin IP. En laboratorio, la solución más simple es `no ip dhcp snooping information option` en el switch.

## Prueba en GNS3

Topología: router con servicio DHCP en `Gi0/0` del switch (trusted) y un VPCS en `Gi0/1`, VLAN 10. En el VPCS:

```text
PC-1> ip dhcp
DD Requested IP: 10.10.10.2/24
```

Luego revisa la binding table en el switch. Para probar el bloqueo, conecta un segundo servidor DHCP en un puerto untrusted: sus OFFER deben aparecer como descartados en `show ip dhcp snooping statistics`. La plantilla del switch está en [[GNS3 - laboratorios de red virtualizados]].

## Notas importantes
- **Todos los puertos son UNTRUSTED por defecto.**
- Solo los puertos hacia servidores DHCP legítimos deben ser `trust`.
- Funciona en switches de **capa 2**, no en routers.
- Requiere que los clientes obtengan IP por DHCP.
- Puertos no confiables bloquean automáticamente respuestas DHCP maliciosas.

> [!info] Base para DAI e IPSG
> La tabla de bindings de DHCP Snooping es la base de **Dynamic ARP Inspection** e **IP Source Guard**. Ver [[Seguridad en la Capa 2]].

---
🔗 Relacionado: [[Servicio DHCP en Cisco]] · [[Seguridad en la Capa 2]] · [[Port Security]] · [[ARP Spoofing y MitM]] · [[GNS3 - laboratorios de red virtualizados]]
