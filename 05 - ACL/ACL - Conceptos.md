---
tags:
  - cisco
  - acl
  - seguridad
aliases:
  - ACL
  - Access Control List
  - Listas de Control de Acceso
---

# Listas de Control de Acceso (ACL) — Conceptos

Una **ACL (Access Control List)** es una lista de reglas que controlan el **tráfico permitido o denegado** a través de un dispositivo de red (router o firewall). Filtra paquetes según IP, protocolo o puerto, aumentando la **seguridad y el control** del tráfico.

## Tipos de ACL

| Tipo | Filtra por | Numeración |
|---|---|---|
| **[[ACL estándar]]** | Solo IP de **origen** | 1–99 y 1300–1999 |
| **[[ACL extendida]]** | Origen, destino, protocolo (TCP/UDP/ICMP) y puerto | 100–199 y 2000–2699 |
| **Nombrada** | Estándar o extendida, con **nombre** personalizado | — |

### ACL Nombrada (ejemplo)

```cisco
ip access-list extended BLOQUEO_HTTP
 deny tcp any any eq 80
 permit ip any any
```

Facilita lectura, edición y mantenimiento frente a las numeradas.

## Clasificación según ubicación
- **Inbound (entrante):** filtra el tráfico **antes** de entrar a la interfaz.
- **Outbound (saliente):** filtra el tráfico que **sale** por la interfaz.

> [!tip] Regla de oro de ubicación
> - **Estándar** → cerca del **destino** (filtra solo por origen, evita bloqueos prematuros).
> - **Extendida** → cerca del **origen** (descarta tráfico no deseado cuanto antes).

## Función principal
- Restringir acceso a redes o subredes.
- Controlar tráfico por protocolo o puerto.
- Mejorar la seguridad del enrutamiento y de [[#NAT y ACL|NAT]].
- Definir qué direcciones pueden ser traducidas o acceder a recursos.

## NAT y ACL

Con **NAT con sobrecarga (PAT)** se usa una ACL estándar para definir qué IPs pueden traducirse:

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

Definir interfaces interna/externa:

```cisco
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface Serial0/0
 ip address 200.10.10.1 255.255.255.252
 ip nat outside
```

**Verificación:** `show ip nat translations` · `show access-lists` · `show running-config`

> [!warning] *Deny any* implícito
> Toda ACL termina con un `deny any` implícito. Si solo pones `permit`, todo lo demás queda bloqueado.

---
🔗 Relacionado: [[ACL estándar]] · [[ACL extendida]] · [[Acceso y autenticación segura en Cisco]]
