---
tags:
  - cisco
  - seguridad
  - capa-2
  - port-security
aliases:
  - Port Security
  - Seguridad de puertos
---

# Cisco Port Security

Característica de seguridad en switches que controla **qué dispositivos pueden conectarse a un puerto** mediante direcciones MAC. Evita accesos no autorizados en la LAN.

## ¿Para qué sirve?
- Restringir dispositivos conectados a un puerto.
- Limitar la cantidad de MAC permitidas.
- Mitigar **MAC Flooding** y conexiones no autorizadas.
- Generar alertas y registros.

## Conceptos clave

| Concepto | Explicación |
|---|---|
| MAC Address | Dirección física única de un dispositivo |
| CAM Table | Tabla donde el switch almacena las MAC |
| Sticky MAC | Aprendizaje automático de MAC autorizadas |
| Err-disabled | Estado en que el puerto queda deshabilitado |
| Access Port | Puerto destinado a un único dispositivo |

## Requisitos
El puerto debe estar en modo `access`, con la interfaz correcta, una cantidad de MAC definida y una acción ante violaciones.

## Configuración

```cisco
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky          ! aprende y guarda MACs
 switchport port-security mac-address 0011.2233.4455  ! MAC estática (opcional)
```

## Modos de violación

| Modo | Bloquea tráfico | Logs | Puerto | Uso |
|---|---|---|---|---|
| **protect** | ✅ | ❌ | activo | continuidad operacional |
| **restrict** | ✅ | ✅ (Syslog + contador) | activo | empresas con monitoreo |
| **shutdown** | ✅ | ✅ | `err-disabled` | alta seguridad |

```cisco
switchport port-security violation restrict
```

## Reactivar un puerto bloqueado

```cisco
shutdown
no shutdown
```

## Ejemplo completo

```cisco
enable
configure terminal
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
end
copy running-config startup-config
```

## Verificación

| Comando | Función |
|---|---|
| `show port-security` | Estado general |
| `show port-security interface Fa0/1` | Configuración detallada |
| `show port-security address` | MAC aprendidas |
| `show mac address-table` | Tabla MAC del switch |

## Contra MAC Flooding
Ataque que inunda la **CAM Table** con MAC falsas para forzar al switch a comportarse como hub (sniffing). Port Security limita las MAC por puerto y bloquea desconocidos.

---
🔗 Tecnologías relacionadas: [[Seguridad en la Capa 2]] · [[DHCP Snooping]] · [[VLAN y enrutamiento Inter-VLAN]] · [[Spanning Tree Protocol (STP)]]
