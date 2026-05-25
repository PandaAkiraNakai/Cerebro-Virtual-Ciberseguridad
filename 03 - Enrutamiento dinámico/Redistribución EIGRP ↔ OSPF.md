---
tags:
  - cisco
  - enrutamiento
  - eigrp
  - ospf
  - redistribución
aliases:
  - Redistribución EIGRP OSPF
---

# Redistribución entre EIGRP y OSPF

Permite que un protocolo comparta las rutas aprendidas con otro, logrando convergencia entre ambos dominios.

## Topología de referencia

```
[ R1 - EIGRP ] —— [ R2 - EIGRP + OSPF ] —— [ R3 - OSPF ]
```

- **R1** usa [[EIGRP]].
- **R2** usa EIGRP **y** [[OSPF]] → punto de redistribución.
- **R3** usa OSPF.

## Configuración del router intermedio (R2)

**EIGRP:**

```cisco
router eigrp 100
 network 192.168.1.0 0.0.0.255
 network 172.16.1.0 0.0.0.255
 no auto-summary
```

**OSPF:**

```cisco
router ospf 1
 network 172.16.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 0
```

**Redistribución cruzada:**

```cisco
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500

router ospf 1
 redistribute eigrp 100 subnets
```

## Parámetros de la métrica (OSPF → EIGRP)

EIGRP no entiende el costo de OSPF, así que hay que **definir la métrica manualmente**:

| Valor | Parámetro | Descripción |
|---|---|---|
| 10000 | Bandwidth | Ancho de banda (Kbps) |
| 100 | Delay | Retardo (décimas de µs) |
| 255 | Reliability | Confiabilidad (0–255; 255 máx.) |
| 1 | Load | Carga (1–255; 1 = baja) |
| 1500 | MTU | Tamaño máximo de unidad de transmisión (bytes) |

> [!important] `subnets` (EIGRP → OSPF)
> Sin la palabra `subnets`, OSPF solo redistribuiría redes *classful*. Con ella se redistribuyen **todas las subredes**.

## Verificación

```cisco
show ip route
show ip protocols
show ip eigrp topology
show ip ospf database
```

---
🔗 Relacionado: [[EIGRP]] · [[OSPF]]
