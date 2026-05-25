---
tags:
  - cisco
  - enrutamiento
  - eigrp
aliases:
  - Enhanced Interior Gateway Routing Protocol
---

# EIGRP — Enhanced Interior Gateway Routing Protocol

Protocolo de enrutamiento dinámico **híbrido** (vector de distancia avanzado) desarrollado por Cisco. Intercambia información de forma eficiente usando una métrica compuesta.

## Características
- **Tipo:** híbrido (vector de distancia avanzado).
- **Algoritmo:** DUAL (Diffusing Update Algorithm) → rutas libres de bucles y convergencia rápida.
- **Métrica compuesta:** Bandwidth, Delay, Reliability, Load, MTU.
- Soporta **VLSM** y **CIDR**; actualizaciones parciales.
- Establece **vecindades** con routers en el mismo **sistema autónomo (AS)**.

## Configuración básica

```cisco
router eigrp 100
 network 192.168.1.0 0.0.0.255
 network 10.0.0.0 0.0.0.255
 no auto-summary
```

| Comando | Significado |
|---|---|
| `router eigrp 100` | Inicia EIGRP con **AS 100**. Solo routers con el mismo AS intercambian rutas. |
| `network ...` | Activa EIGRP en las interfaces de esa red (envía paquetes Hello). |
| `no auto-summary` | Desactiva la sumarización automática por clase → permite anunciar subredes (necesario con VLSM o redes discontiguas). |

## Segundo router (ejemplo)

```cisco
router eigrp 100
 network 10.0.0.0 0.0.0.255
 network 172.16.1.0 0.0.0.255
 no auto-summary
```

## Verificación

```cisco
show ip route eigrp
show ip eigrp neighbors
show ip eigrp topology
show ip protocols
```

## Notas y buenas prácticas
- Todos los routers que formen vecindad deben usar el **mismo AS**.
- `no auto-summary` debe estar habilitado en configuraciones modernas.
- Revisa rutas de respaldo (*feasible successors*) con `show ip eigrp topology`.
- Documenta redes anunciadas e interfaces participantes.

---
🔗 Relacionado: [[OSPF]] · [[Redistribución EIGRP ↔ OSPF]]
