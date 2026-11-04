---
tags:
  - cisco
  - enrutamiento
  - ospf
  - link-state
  - seguridad
aliases:
  - Open Shortest Path First
  - OSPFv2
  - Wildcard mask
  - Autenticación OSPF
---

# OSPF — Open Shortest Path First

Protocolo de enrutamiento dinámico de tipo **link-state** (estado de enlace). Cada router guarda una copia del mapa completo de la red (la **LSDB**) y calcula por su cuenta el camino más corto a cada destino con el algoritmo de Dijkstra.

## Características

| Característica | Valor |
|---|---|
| Tipo | Link-State |
| Algoritmo | Dijkstra / SPF (Shortest Path First) |
| Métrica | Costo, según el ancho de banda |
| Diseño | Jerárquico, por áreas; el **área 0** es el backbone |
| Distancia administrativa (Cisco) | 110 |

### Cálculo del costo

```text
Costo = ancho de banda de referencia / ancho de banda de la interfaz
Referencia por defecto = 100 Mbps
```

| Interfaz | Cálculo | Costo |
|---|---|---|
| Ethernet (10 Mbps) | 100 / 10 | 10 |
| FastEthernet (100 Mbps) | 100 / 100 | 1 |
| GigabitEthernet (1 Gbps) | 100 / 1000 | 1 (mínimo) |

> [!warning] FastEthernet y Gigabit empatan
> Con la referencia por defecto ambos cuestan 1. En redes modernas ajusta `auto-cost reference-bandwidth 10000` con **el mismo valor en todos los routers** del dominio.

## Conceptos clave

| Concepto | Qué es | Ejemplo |
|---|---|---|
| Process ID | Número local del proceso; no necesita coincidir entre routers | `router ospf 1` |
| Router ID | Identificador único del router en el dominio (formato IPv4) | `1.1.1.1` |
| Área | Grupo lógico de redes que comparten la misma LSDB | `area 0` |
| LSDB | Base de datos con el mapa de la topología | `show ip ospf database` |
| LSA | Anuncio de estado de enlace; las piezas de la LSDB | — |
| Vecino | Router OSPF directamente conectado | `show ip ospf neighbor` |

### Estados de una vecindad

```text
DOWN → INIT → 2-WAY → EXSTART → EXCHANGE → LOADING → FULL
```

| Estado | Significado |
|---|---|
| DOWN | No se han recibido Hellos |
| INIT | Llegó un Hello, pero el vecino aún no nos reconoce |
| 2-WAY | Ambos se reconocen; en redes multiacceso se elige DR/BDR |
| EXSTART / EXCHANGE | Se negocia y se intercambia el resumen de la LSDB |
| LOADING | Se piden los LSA que faltan |
| **FULL** | LSDB sincronizada; vecindad operativa |

> [!info] Autenticación distinta = vecino invisible
> Si la autenticación no coincide, los Hellos se descartan y la vecindad **no sale de DOWN**: el vecino ni siquiera aparece.

## Wildcard mask

Indica **qué bits de la dirección se comparan** (`0`) y cuáles se ignoran (`1`). En OSPF decide en qué interfaces se activa el protocolo con el comando `network`. Es la inversa lógica de la máscara de subred.

**Cálculo rápido:** `255.255.255.255 − máscara de subred`.

```text
/26 → 255.255.255.255 − 255.255.255.192 = 0.0.0.63
/30 → 255.255.255.255 − 255.255.255.252 = 0.0.0.3
/20 → 255.255.255.255 − 255.255.240.0   = 0.0.15.255
```

| Prefijo | Máscara | Wildcard | Hosts útiles |
|---|---|---|---|
| /8 | 255.0.0.0 | 0.255.255.255 | 16.777.214 |
| /16 | 255.255.0.0 | 0.0.255.255 | 65.534 |
| /20 | 255.255.240.0 | 0.0.15.255 | 4.094 |
| /24 | 255.255.255.0 | 0.0.0.255 | 254 |
| /25 | 255.255.255.128 | 0.0.0.127 | 126 |
| /26 | 255.255.255.192 | 0.0.0.63 | 62 |
| /27 | 255.255.255.224 | 0.0.0.31 | 30 |
| /28 | 255.255.255.240 | 0.0.0.15 | 14 |
| /29 | 255.255.255.248 | 0.0.0.7 | 6 |
| /30 | 255.255.255.252 | 0.0.0.3 | 2 |
| /32 | 255.255.255.255 | 0.0.0.0 | 1 (host) |

Casos especiales:

| Wildcard | Efecto |
|---|---|
| `0.0.0.0` | Coincide solo la IP exacta: activa OSPF en **una** interfaz (la forma más precisa) |
| `255.255.255.255` | Coincide cualquier IP: activa OSPF en **todas** (no recomendado) |

> [!example] Comprobación
> `network 10.0.0.0 0.0.255.255 area 0` activa OSPF en `10.0.5.1` y `10.0.200.9`, pero **no** en `10.1.0.1` (el segundo octeto no coincide).

## Configuración básica

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1   ! LAN de usuarios: se anuncia, pero sin Hellos
```

> [!tip] `network` no anuncia redes
> Activa OSPF en las interfaces que coinciden; después OSPF anuncia la red configurada en esas interfaces.

Alternativa sin wildcard, desde la interfaz:

```cisco
interface GigabitEthernet0/0
 ip ospf 1 area 0
```

## Autenticación OSPF

Sin autenticación, cualquier equipo conectado al segmento puede formar vecindad e **inyectar rutas falsas**.

| Tipo | Nombre | Seguridad | Comentario |
|---|---|---|---|
| 0 | Null | Ninguna | Por defecto |
| 1 | Contraseña simple | Muy baja | Viaja en texto plano; se ve en Wireshark |
| 2 | MD5 | Media | Viaja un hash del mensaje + clave, no la clave |
| — | HMAC-SHA (key chain) | Alta | IOS 15.4(1)T o superior (RFC 5709) |

```cisco
! Tipo 1 — contraseña simple (evitar)
interface GigabitEthernet0/0
 ip ospf authentication
 ip ospf authentication-key MiClavePlana

! Tipo 2 — MD5 (Key ID y clave iguales en ambos extremos)
interface GigabitEthernet0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 ClaveSecreta

! HMAC-SHA con key chain (recomendado si el IOS lo soporta)
key chain OSPF-KEYS
 key 1
  key-string ClaveSegura
  cryptographic-algorithm hmac-sha-256
interface GigabitEthernet0/0
 ip ospf authentication key-chain OSPF-KEYS
```

> [!warning] MD5 no cifra
> Solo da autenticación e integridad. El contenido OSPF sigue visible en una captura; lo que no se ve es la clave.

### Por interfaz o por área

```cisco
! Por área: el requisito aplica a todas las interfaces del área
router ospf 1
 area 0 authentication message-digest
! La clave igual se pone en cada interfaz
interface GigabitEthernet0/0
 ip ospf message-digest-key 1 md5 ClaveSecreta
```

| Característica | Por interfaz | Por área |
|---|---|---|
| Alcance | Una interfaz | Todas las del área |
| Dónde se activa | `interface` | `router ospf` |
| Dónde va la clave | En la interfaz | En cada interfaz |
| Uso | Control fino por enlace | Política uniforme |

La configuración de la interfaz **gana** sobre la del área. Para excluir una interfaz: `ip ospf authentication null`.

## Ejemplo: dos routers con MD5

```text
 10.0.0.0/24            192.168.1.0/24            172.16.1.0/24
[LAN R1]── R1 G0/1  R1 G0/0 ═══(MD5)═══ R2 G0/0  R2 G0/1 ──[LAN R2]
                       .1                 .2
           RID 1.1.1.1                    RID 2.2.2.2
```

```cisco
! R1
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1
interface GigabitEthernet0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 ClaveSecreta

! R2
router ospf 1
 router-id 2.2.2.2
 network 192.168.1.0 0.0.0.255 area 0
 network 172.16.1.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/1
interface GigabitEthernet0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 ClaveSecreta
```

Resultado esperado:

```text
R1# show ip ospf neighbor
Neighbor ID  Pri  State     Dead Time  Address      Interface
2.2.2.2        1  FULL/DR   00:00:35   192.168.1.2  GigabitEthernet0/0
```

## Requisitos de adyacencia

| Parámetro | Si no coincide… |
|---|---|
| Área | No hay vecindad |
| Subred y máscara | No hay vecindad |
| Timers Hello / Dead | No hay vecindad |
| Tipo de autenticación, clave y Key ID | Los Hellos se descartan |
| Tipo de red OSPF | Vecindad inestable o rutas incorrectas |
| MTU | Se queda en EXSTART / EXCHANGE |
| Router ID | Debe ser **distinto**; repetido genera conflicto |

## Verificación

```cisco
show ip ospf neighbor                      ! vecinos (se espera FULL)
show ip route ospf                         ! rutas aprendidas (O)
show ip ospf database                      ! LSDB
show ip protocols                          ! Router ID, redes y áreas
show ip ospf interface GigabitEthernet0/0  ! área, timers, tipo de red y autenticación
```

Con MD5 activo, `show ip ospf interface` muestra `Message digest authentication enabled` y `Youngest key id is 1`.

## Diagnóstico

```text
¿Aparece el vecino?
 ├─ NO → interfaz up/up · ping · área/subred/timers · autenticación · ¿passive por error?
 └─ SÍ → ¿está en FULL?
          ├─ EXSTART/EXCHANGE → revisa MTU
          ├─ 2-WAY → normal entre DROTHER en redes multiacceso
          └─ FULL → revisa show ip route ospf
```

```cisco
debug ip ospf adj
undebug all        ! apágalo al terminar
```

| Mensaje | Causa |
|---|---|
| `Mismatched Authentication type` | Un extremo usa MD5 y el otro no, o usa tipo 1 |
| `Mismatched Authentication Key - Message Digest Key 1` | La clave no coincide |
| `No message digest key 1 on interface` | El Key ID no coincide o falta |

## Área 0 y Router ID

- En un diseño multiárea **todas las áreas se conectan al área 0**; el tráfico entre áreas pasa por el backbone. Los routers que unen un área con el backbone son **ABR** (Area Border Router). Si un área no puede tocar el área 0 físicamente, se usa un **virtual link**.
- Si no fijas el Router ID, IOS lo elige así: valor de `router-id` → IP más alta de una **loopback** activa → IP más alta de una interfaz física activa.
- Tras cambiar el Router ID aplica `clear ip ospf process` (reinicia las vecindades).

## Buenas prácticas

- Router ID fijos y únicos.
- Wildcard `0.0.0.0` con la IP exacta, o `ip ospf <proceso> area <área>` en la interfaz.
- `passive-interface` hacia usuarios finales.
- Autenticación en todos los enlaces entre routers: HMAC-SHA si se puede, si no MD5; nunca la simple.
- Mismo `auto-cost reference-bandwidth` en todos los routers si hay enlaces de 1 Gbps o más.
- Documentar Router ID, áreas, redes, interfaces y parámetros de autenticación.

---
🔗 Relacionado: [[EIGRP]] · [[Redistribución EIGRP ↔ OSPF]] · [[Direcciones IP estáticas y rutas estáticas]] · [[ACL - Conceptos]]
