---
tags:
  - monitoreo
  - snmp
  - cisco
  - seguridad
aliases:
  - SNMPv3
  - SNMP v3 Cisco
  - snmpwalk
---

# SNMPv3 en Cisco

SNMPv2c se autentica con una **comunidad** que viaja en texto plano: quien capture el tráfico la lee y puede consultar el equipo. **SNMPv3** agrega usuarios con autenticación (HMAC) y cifrado (AES) mediante el modelo USM y control de acceso por vistas (VACM). Usa UDP 161 para consultas y UDP 162 para traps.

| Aspecto | SNMPv2c | SNMPv3 |
|---|---|---|
| Identidad | Comunidad compartida | Usuario |
| Integridad | No | HMAC (SHA o MD5) |
| Cifrado | No | AES o DES |
| Control de acceso | ACL + RO/RW | Grupos, vistas y ACL |

## Niveles de seguridad

| Nivel | Palabra en IOS | Autentica | Cifra |
|---|---|---|---|
| noAuthNoPriv | `noauth` | No | No |
| authNoPriv | `auth` | Sí | No |
| **authPriv** | `priv` | Sí | **Sí** |

Para monitoreo real usa **authPriv**.

## Configuración en el router

```cisco
! 1. ACL: solo el servidor de monitoreo puede consultar
ip access-list standard ACL-SNMP
 permit host 192.168.1.100

! 2. Vista: qué parte del árbol MIB se expone
snmp-server view VISTA-RO iso included

! 3. Grupo: nivel priv, lectura con la vista, filtrado por la ACL
snmp-server group GRUPO-RO v3 priv read VISTA-RO access ACL-SNMP

! 4. Usuario dentro del grupo, con clave de autenticación y de cifrado
snmp-server user zabbix GRUPO-RO v3 auth sha ClaveAuth2026 priv aes 128 ClavePriv2026

! 5. Traps v3 hacia el servidor
snmp-server host 192.168.1.100 version 3 priv zabbix
snmp-server enable traps

! Datos informativos
snmp-server location "Sala de servidores"
snmp-server contact "noc@ejemplo.local"
```

> [!info] El usuario no aparece en la running-config
> IOS guarda los usuarios SNMPv3 asociados al *engine ID* local y no los muestra en `show running-config`. Revísalos con `show snmp user`. Si cambias el engine ID, debes volver a crearlos.

> [!tip] Si no puedes salir de v2c
> Usa una comunidad propia (nunca `public`), en solo lectura y amarrada a una ACL: `snmp-server community MiComunidadRO RO ACL-SNMP`.

## Verificación en el router

```cisco
show snmp user
show snmp group
show snmp view
show snmp engineID
show snmp            ! contadores de paquetes y errores
```

## Prueba desde Linux

```bash
sudo apt install snmp
snmpwalk -v3 -l authPriv -u zabbix -a SHA -A ClaveAuth2026 -x AES -X ClavePriv2026 \
  192.168.1.1 1.3.6.1.2.1.1.1.0          # sysDescr
```

## Alta en Zabbix

En el host, agrega una interfaz **SNMP** con versión **SNMPv3** y los mismos datos del router:

| Campo | Valor |
|---|---|
| Security name | `zabbix` |
| Security level | authPriv |
| Authentication protocol / passphrase | SHA1 / clave de autenticación |
| Privacy protocol / passphrase | AES128 / clave de cifrado |

Asocia la plantilla **Cisco IOS by SNMP** para obtener interfaces, CPU, memoria y estado.

## OID útiles

| OID | Objeto |
|---|---|
| `1.3.6.1.2.1.1.1.0` | sysDescr: descripción del equipo |
| `1.3.6.1.2.1.1.3.0` | sysUpTime: tiempo encendido |
| `1.3.6.1.2.1.2.2.1.8` | ifOperStatus: estado de cada interfaz |
| `1.3.6.1.2.1.31.1.1.1.6` | ifHCInOctets: tráfico de entrada (64 bits) |
| `1.3.6.1.2.1.31.1.1.1.10` | ifHCOutOctets: tráfico de salida (64 bits) |
| `1.3.6.1.4.1.9.9.109.1.1.1.1.8` | cpmCPUTotal5minRev: CPU de Cisco a 5 minutos |

## Problemas frecuentes

| Síntoma | Causa probable |
|---|---|
| `Timeout` | La ACL no incluye al servidor, firewall en UDP 161 o IP equivocada |
| `Unknown user name` | Usuario mal escrito o borrado al cambiar el engine ID |
| `Authentication failure` | Clave o algoritmo de autenticación distinto |
| `Decryption error` | Clave o algoritmo de cifrado distinto |
| `authorizationError` o sin datos | La vista no incluye ese OID |
| Las traps no llegan | El servidor no escucha UDP 162 o falta `snmp-server enable traps` |

---
🔗 Relacionado: [[Zabbix y SNMP en Cisco]] · [[Zabbix 7.4 con MySQL]] · [[Zabbix y SNMP en MikroTik]] · [[ACL estándar]] · [[Wireshark]]
