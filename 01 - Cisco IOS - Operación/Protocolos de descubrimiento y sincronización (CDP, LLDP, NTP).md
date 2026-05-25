---
tags:
  - cisco
  - operación
  - descubrimiento
  - ntp
aliases:
  - CDP
  - LLDP
  - NTP
---

# Protocolos de descubrimiento y sincronización (CDP, LLDP, NTP)

## 1. CDP (Cisco Discovery Protocol)

Protocolo de **capa 2 propietario de Cisco**, activo por defecto. Comparte información entre dispositivos Cisco conectados directamente.

**Uso:** detectar vecinos Cisco (routers, switches, telefonía IP); obtener hostname, IP, capacidades, plataforma e IDs de interfaces; reconstruir topología física.

| Acción | Comando (modo) |
|---|---|
| Habilitar globalmente | `cdp run` (config) |
| Deshabilitar globalmente | `no cdp run` (config) |
| Habilitar en interfaz | `cdp enable` (config-if) |
| Deshabilitar en interfaz | `no cdp enable` (config-if) |

**Verificación:** `show cdp neighbors` · `show cdp neighbors detail` · `show cdp traffic`

## 2. LLDP (Link Layer Discovery Protocol — IEEE 802.1AB)

Descubrimiento de **capa 2 estándar y abierto**. Alternativa neutral a CDP para entornos **multi-vendor** (Cisco, HP, Juniper, etc.).

| Acción | Comando (modo) |
|---|---|
| Habilitar globalmente | `lldp run` (config) |
| Deshabilitar globalmente | `no lldp run` (config) |
| Habilitar envío en interfaz | `lldp transmit` (config-if) |
| Habilitar recepción en interfaz | `lldp receive` (config-if) |

**Verificación:** `show lldp neighbors` · `show lldp neighbors detail` · `show lldp traffic`

> [!note] Seguridad
> En equipos críticos suele **deshabilitarse** CDP/LLDP (`no cdp run` / `no lldp run`) para no filtrar información. Ver [[Seguridad en la Capa 2]].

## 3. NTP (Network Time Protocol)

Sincroniza los relojes de los dispositivos vía red usando **UDP 123**.

**Importancia:** estampas de tiempo coherentes en logs (Syslog), validación de certificados y autenticación basada en tiempo.

### Stratum (estrato)
- **Stratum 0**: fuentes de alta precisión (relojes atómicos, GPS).
- **Stratum 1**: servidores conectados directo a Stratum 0.
- **Stratum n**: cada salto suma un nivel (máximo 15).

| Acción | Comando (modo) |
|---|---|
| Servidor (maestro) | `ntp master <estrato>` (config) |
| Cliente | `ntp server <IP>` (config) |
| Actualizar reloj de hardware | `ntp update-calendar` (config) |
| Hora manual | `clock set hh:mm:ss <día> <mes> <año>` (Priv. EXEC) |

**Verificación:** `show ntp status` · `show ntp associations` · `show clock detail`

> [!tip] Guarda los cambios con `copy running-config startup-config`.

---
🔗 Relacionado: [[Configuración básica de equipos Cisco]] · [[Seguridad en la Capa 2]]
