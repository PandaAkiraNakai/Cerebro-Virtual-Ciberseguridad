---
tags:
  - monitoreo
  - zabbix
  - snmp
  - cisco
aliases:
  - SNMPv2 en Cisco para Zabbix
---

# Zabbix y SNMP en Cisco

**Configuración de SNMPv2c en un router Cisco para que un servidor de monitoreo Zabbix pueda consultar su estado y recibir traps. Al terminar, el router queda listo para entregar información (interfaces, CPU, memoria) y notificar eventos hacia Zabbix.**

## Requisitos previos

- Acceso administrativo a la CLI del router Cisco.
- Un servidor Zabbix operativo listo para recibir consultas y traps SNMP.
- Conocimientos básicos de Cisco IOS y de la interfaz de Zabbix.

## Configuración en el router Cisco

```cisco
enable
configure terminal

! Comunidad SNMP de solo lectura (RO)
snmp-server community <comunidad> RO

! (Opcional) Ubicación y contacto administrativo
snmp-server location "Data Center 1"
snmp-server contact "admin@example.com"

! Host Zabbix que recibirá las traps (IP + versión + comunidad)
snmp-server host <ip_servidor_zabbix> version 2c <comunidad>

! Habilitar envío de traps
snmp-server enable traps

end
write memory
```

> [!warning] Nombre de comunidad
> No uses la comunidad por defecto `public`. Elige un nombre propio y difícil de adivinar; en SNMPv2c la comunidad viaja en texto plano, así que limita además el acceso SNMP con ACL hacia la IP del servidor Zabbix.

## Qué hace cada comando

| Comando | Función |
| :--- | :--- |
| `enable` | Entra al modo privilegiado. |
| `configure terminal` | Entra al modo de configuración global. |
| `snmp-server community <nombre> RO` | Define la comunidad SNMP en solo lectura. |
| `snmp-server location` | Ubicación física del equipo (informativo). |
| `snmp-server contact` | Contacto del administrador (informativo). |
| `snmp-server host <ip> version 2c <comunidad>` | Servidor que recibe las traps, versión y comunidad. |
| `snmp-server enable traps` | Habilita el envío de traps. |
| `write memory` | Guarda la configuración en NVRAM. |

> [!tip] Lado Zabbix
> En Zabbix la IP, la versión (SNMPv2c) y el nombre de comunidad deben coincidir exactamente con lo configurado en el router. Asocia una plantilla SNMP apropiada al host para recibir los gráficos.

---
📎 Guías: ![[Zabbix-SNMP-Cisco.pdf]]

🔗 Relacionado: [[Zabbix y SNMP en MikroTik]] · [[Splunk Enterprise]] · [[Wireshark]]
