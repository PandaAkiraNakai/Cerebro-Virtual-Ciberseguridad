---
tags:
  - cisco
  - enrutamiento
  - ospf
  - link-state
aliases:
  - Open Shortest Path First
---

# OSPF — Open Shortest Path First

Protocolo de enrutamiento dinámico de tipo **link-state** (estado de enlace). Determina la mejor ruta hacia cada red dentro de un sistema autónomo calculando el **costo** según el ancho de banda.

## Características
- **Tipo:** Link-State.
- **Algoritmo:** Dijkstra (SPF — Shortest Path First).
- **Métrica:** costo (basado en ancho de banda).
- **Áreas jerárquicas:** el **área 0** es el backbone; permite segmentar redes grandes para optimizar CPU/memoria.

## Configuración básica

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.255 area 0
```

| Comando | Significado |
|---|---|
| `router ospf 1` | Inicia el proceso OSPF con ID **1** (local al router). |
| `router-id 1.1.1.1` | Identificador único en el dominio OSPF. Si no se fija, toma la IP más alta de las interfaces activas. |
| `network ... area 0` | Activa OSPF en las interfaces de esa red y las asigna al área indicada. |

## Segundo router (ejemplo)

```cisco
router ospf 1
 router-id 2.2.2.2
 network 192.168.1.0 0.0.0.255 area 0
 network 172.16.1.0 0.0.0.255 area 0
```

## Verificación

```cisco
show ip route ospf
show ip ospf neighbor
show ip ospf database
show ip protocols
```

## Notas y buenas prácticas
- El **router-id** es esencial para identificar el router en la LSDB; úsalo **fijo y único**.
- Toda interfaz OSPF debe pertenecer a un área; el **área 0** debe existir en topologías jerárquicas.
- Si hay más de un área, debe haber conectividad (directa o *virtual link*) hacia el área 0.
- Evita sobrecargar el área 0; verifica vecindades y propagación antes de redistribuir.

---
🔗 Relacionado: [[EIGRP]] · [[Redistribución EIGRP ↔ OSPF]] · [[Direcciones IP estáticas y rutas estáticas]]
