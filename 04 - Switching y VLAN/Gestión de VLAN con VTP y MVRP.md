---
tags:
  - cisco
  - switching
  - vlan
  - vtp
aliases:
  - VTP
  - VLAN Trunking Protocol
  - GVRP
  - MVRP
---

# Gestión de VLAN con VTP y MVRP

Sin un protocolo de propagación, cada VLAN se crea **a mano en cada switch**. VTP, GVRP y MVRP distribuyen la base de VLAN desde un punto. La propagación **solo viaja por troncales 802.1Q** (ver [[VLAN y enrutamiento Inter-VLAN]]).

| Protocolo | Estándar | Estado |
|---|---|---|
| VTP | Cisco | Usar con precaución |
| GVRP | IEEE 802.1Q (GARP) | Obsoleto |
| MVRP | IEEE 802.1ak (MRP) | Reemplazo estándar de GVRP |

## VTP — VLAN Trunking Protocol

### Modos

| Modo | Crea/borra VLAN | Sincroniza | Reenvía anuncios | Guarda en `vlan.dat` |
|---|---|---|---|---|
| Server | Sí | Sí | Sí | Sí |
| Client | No | Sí | Sí | v3 sí · v1-v2 no |
| Transparent | Solo locales | No | Sí | Sí |
| Off (v3) | Solo locales | No | No | Sí |

Dos switches solo intercambian información si comparten **nombre de dominio** (distingue mayúsculas), **versión**, **contraseña** y un **troncal**.

### Versiones

| Característica | v1 | v2 | v3 |
|---|---|---|---|
| VLAN normales (1-1005) | Sí | Sí | Sí |
| VLAN extendidas (1006-4094) | No | No | Sí |
| Transparent reenvía sin revisar versión | No | Sí | Sí |
| Primary Server | No | No | Sí |
| Modo Off | No | No | Sí |
| Contraseña oculta | No | No | Sí |
| Propaga instancias MST | No | No | Sí |

Si usas VTP, que sea **v3**.

### Configuración

```cisco
! Server
vtp domain EMPRESA
vtp version 2
vtp mode server
vtp password ClaveVTP

! Client
vtp domain EMPRESA
vtp version 2
vtp mode client
vtp password ClaveVTP

! Transparent
vtp mode transparent

! VTPv3 con Primary Server
vtp domain EMPRESA
vtp version 3
vtp mode server
vtp password ClaveVTP hidden
end
vtp primary vlan          ! en modo EXEC, no en configuración global
```

## El peligro del revision number

Cada cambio en un Server sube el **Configuration Revision Number**, y los switches **aceptan siempre la base con el número más alto**, aunque esté vacía.

```text
SW de laboratorio            Producción
dominio EMPRESA, rev 87      rev 12, VLAN 10-50
solo VLAN 1
      └───── troncal ─────►  87 > 12 → adopta la base vacía
                             VLAN 10-50 borradas, puertos de acceso inactivos
```

> [!danger] Caída total por un switch reciclado
> En v1/v2 **incluso un Client** puede propagar su base si su revisión es mayor. Resetea la revisión de todo switch antes de conectarlo.

Resetear la revisión a 0:

```cisco
! Opción 1: pasar por Transparent
vtp mode transparent
vtp mode client
! Opción 2: cambiar el dominio y volver
vtp domain RESET
vtp domain EMPRESA
! Comprobar
show vtp status        ! Configuration Revision : 0
```

Buenas prácticas:

- Resetear la revisión de cada switch nuevo o reciclado.
- Contraseña de dominio siempre.
- VTPv3: solo el Primary Server puede modificar.
- Muchas redes dejan casi todo en Transparent u Off y gestionan las VLAN con automatización (ver [[Ansible]]).

## GVRP y MVRP

**GVRP** es el equivalente estándar de VTP: cada puerto **declara** las VLAN que necesita con mensajes Join de GARP, y se registran a lo largo de los troncales. No tiene revision number, así que no existe el borrado masivo. Está obsoleto por lento y pesado.

**MVRP** (802.1ak, sobre MRP) lo reemplaza y es la opción multivendor actual. En las plataformas Cisco que lo soportan:

```cisco
mvrp global
interface GigabitEthernet0/1
 switchport mode trunk
 mvrp
```

| Característica | VTP v1/v2 | VTP v3 | GVRP | MVRP |
|---|---|---|---|---|
| Estándar abierto | No | No | Sí | Sí |
| VLAN extendidas | No | Sí | Sí | Sí |
| Riesgo de borrado | Alto | Bajo | No | No |
| Estado | Legado | Vigente | Obsoleto | Vigente |

## Verificación

```cisco
show vtp status
show vtp password
show vtp counters
show vlan brief
show interfaces trunk
```

## Errores frecuentes

| Síntoma | Causa |
|---|---|
| No sincroniza | Dominio con distinta capitalización (`Empresa` ≠ `EMPRESA`) o sin troncal |
| `MD5 digest checksum mismatch` | Contraseña distinta |
| `VTP VLAN configuration not allowed when device is in CLIENT mode.` | Intentaste crear la VLAN en un Client |
| VLAN borradas en toda la red | Switch conectado sin resetear la revisión |

---
🔗 Relacionado: [[VLAN y enrutamiento Inter-VLAN]] · [[Spanning Tree Protocol (STP)]] · [[EtherChannel con LACP]] · [[FHRP - HSRP, VRRP y GLBP]] · [[Seguridad en la Capa 2]]
