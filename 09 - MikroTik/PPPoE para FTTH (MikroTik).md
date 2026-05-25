---
tags:
  - mikrotik
  - pppoe
  - ftth
  - isp
aliases:
  - PPPoE MikroTik
  - MikroTik FTTH
---

# PPPoE para FTTH (MikroTik)

Configurar un router **MikroTik** con servicio **PPPoE** para crear usuarios y asignar perfiles de navegación con ancho de banda determinado. Ideal para redes **FTTH** o montar tu propio **ISP** (con servicios como IPTV o telefonía IP).

## Protocolo PPPoE

**PPPoE** (Point-to-Point Protocol over Ethernet) establece conexiones de banda ancha sobre Ethernet (DSL, cable, fibra). Características:

- **Autenticación** por usuario/contraseña antes de acceder a la red.
- **Sesiones individuales** por usuario → control de acceso granular.
- **Eficiencia** (overhead mínimo) y **seguridad** (datos cifrados).
- **Control de sesiones** por el ISP (políticas de uso, gestión de tráfico).
- Compatible con **IPv6** y ampliamente adoptado.

## Orden de configuración

> [!abstract] Lógica
> Entregar internet con distintos anchos de banda. El orden importa: primero la WAN, luego el servicio PPPoE.

**Configuración WAN:**
1. Seleccionar la interfaz física hacia el proveedor.
2. Asignar IP (DHCP automático o manual).
3. Crear ruta por defecto.
4. Configurar NAT.

**Servicio PPPoE:**
- Crear interfaz **Bridge** y agregar puertos físicos (LAN).
- Asignar IP a la interfaz Bridge.
- Crear **Pool** de IPs para los usuarios PPPoE.
- Crear el **servidor PPPoE**.
- Crear **perfiles** de navegación y asignarles el Pool.
- Crear **usuarios** y asignarles los perfiles.

## Pasos en Winbox

### WAN — DHCP automático
`IP → DHCP Client → [+]` → seleccionar la interfaz WAN (ej. `eth1`).

### NAT (masquerade)
`IP → Firewall → NAT → [+]` → interfaz de salida WAN (`eth1`) → pestaña *Action* → **masquerade**.

### Interfaz Bridge (LAN)
`Bridge → [+]` → nombre → *Apply*. Luego `Bridge → Ports → [+]` → agregar la interfaz LAN.

### IP del Bridge
`IP → Address → [+]` → IP + máscara → seleccionar el bridge creado.

### Pool de IPs
`IP → Pool → [+]` → nombre + rango, ej. `192.168.3.2-192.168.3.254`.

### Servidor PPPoE
`PPP → PPPoE Server → [+]` → nombre del servicio → interfaz de salida (LAN) → marcar **one session per host**.

### Perfiles (planes de navegación)
`PPP → Profiles → [+]` → nombre → *Local Address* (IP de salida LAN) → asignar el Pool → DNS → en **Limits** asignar el ancho de banda del perfil.

### Usuarios PPPoE
`PPP → Secrets → [+]` → usuario + contraseña → asignar Pool y perfil creados.

---
📎 Guía completa con capturas: ![[MikroTik-PPPoE-FTTH.pdf]]

🔗 Relacionado: [[Capacity Planning y QoS]] · [[Servicio DHCP en Cisco]]
