---
tags:
  - auditoria
  - riesgos
  - telecomunicaciones
  - proyectos
aliases:
  - Riesgos en telecomunicaciones
  - Registro de riesgos
  - PPDIOO riesgos
---

# Análisis de riesgos en proyectos de telecomunicaciones

En un proyecto de red el riesgo no es solo un atacante: es **todo lo que puede hacer fallar el proyecto**. Equipos que fallan, interferencias, permisos atrasados, proveedores que no entregan o requisitos que cambian. Se cuantifica igual que en [[Cálculo de riesgos (Probabilidad × Impacto)|R = P × I]].

## Las cuatro etapas

1. **Identificar:** ¿qué puede fallar?
2. **Evaluar:** P × I para cada riesgo.
3. **Priorizar:** ordenar por nivel.
4. **Mitigar:** plan para bajar P, I o ambos.

| Fase PPDIOO | Rol en el análisis |
|---|---|
| Preparar | Riesgos de negocio, presupuesto y alcance |
| Planear | Riesgos técnicos, de recursos y de plazos |
| Diseñar | Mitigar por diseño: redundancia, holguras, margen de enlace |
| Implementar | Terreno, proveedores, pruebas de aceptación |
| Operar | Riesgo residual: fallas y SLA |
| Optimizar | Reevaluar el registro con datos reales |

## Categorías

| Categoría | Ejemplos |
|---|---|
| Técnico / hardware | Falla de OLT, switch core, fuentes, transceptores |
| Radioeléctrico | Interferencia, obstrucción de la zona de Fresnel, desvanecimiento por lluvia |
| Interoperabilidad | ONT incompatible con la OLT, firmware, protocolos propietarios |
| Requisitos / alcance | Cambios del cliente, nuevas zonas, más ancho de banda |
| Proveedores / logística | Atraso de equipos, quiebre de stock de fibra |
| Obra civil / terreno | Permisos municipales, uso de postes, servidumbres, canalizaciones |
| Regulatorio | Autorizaciones y concesiones del regulador, normativa de torres y espectro |
| Recursos humanos | Falta de técnicos certificados (fusión, radioenlaces), rotación |
| Ambiental | Clima, viento, sismos, cortes eléctricos en sitios remotos |

## De dónde salen los valores

| Dato | Cómo obtenerlo | Ejemplo |
|---|---|---|
| Proyectos anteriores | Informes de cierre, lecciones aprendidas | 4 de 8 despliegues FTTH se atrasaron por permisos |
| Confiabilidad | Fichas técnicas (MTBF, AFR), garantías | OLT con MTBF de 200.000 h |
| Terreno | Site survey, analizador de espectro, línea de vista | 7 redes vecinas en el canal de 5,8 GHz |
| Proveedores | Cotizaciones, *lead time*, historial | ONT con 10-14 semanas de entrega |
| Trámites | Tiempos históricos | Permiso de postes: 45-90 días |
| Operación | Tickets del NOC, registros de fallas | 3 cortes de energía al mes en el sitio |

## Escalas de ejemplo

| P | Nivel | Frecuencia en proyectos similares |
|---|---|---|
| 1 | Muy baja | Menos del 5 %, sin evidencia en terreno |
| 2 | Baja | 5-20 % |
| 3 | Media | 20-40 %, condiciones de riesgo presentes |
| 4 | Alta | 40-70 %, evidencia directa en survey o historial |
| 5 | Muy alta | Más del 70 % o ya ocurriendo |

El impacto es **multidimensional**: evalúa cada columna y quédate con la más alta.

| I | Plazo | Costo | Servicio | Contractual / regulatorio |
|---|---|---|---|---|
| 1 | < 1 semana | < 2 % | Sin efecto | Ninguno |
| 2 | 1-2 semanas | 2-5 % | Pruebas internas | Ninguno |
| 3 | 2-4 semanas | 5-10 % | < 500 clientes o calidad degradada | Hito en riesgo |
| 4 | 1-3 meses | 10-20 % | 500-5.000 clientes sin servicio | Multas por SLA |
| 5 | > 3 meses | > 20 % | > 5.000 clientes o core caído | Incumplimiento normativo, proyecto inviable |

## Redactar y clasificar

Redacta cada riesgo como **causa → evento → efecto**:

> "Debido a la saturación de la banda de 5,8 GHz, puede haber interferencia en el radioenlace troncal, lo que provocaría pérdida de paquetes en 800 clientes."

| Riesgo | Rango | Acción en el proyecto |
|---|---|---|
| Bajo | 1-5 | Aceptar y monitorear |
| Medio | 6-9 | Mitigar en el diseño |
| Alto | 10-16 | Mitigar antes de implementar, con responsable |
| Crítico | 17-25 | No avanzar de fase sin mitigación aprobada |

## Estrategias de tratamiento

| Estrategia | Qué hace | Ejemplo |
|---|---|---|
| Evitar | Elimina la causa | Pasar de 5,8 GHz libre a una banda licenciada |
| Mitigar | Baja P o I | Redundancia 1+1, UPS, repuestos, holgura en el cronograma |
| Transferir | Traspasa el riesgo | Multas contractuales al proveedor, seguro, subcontratar la obra civil |
| Aceptar | Se asume con monitoreo | Riesgos bajos con plan de contingencia |

Después de tratar, **recalcula**: eso es el riesgo residual.

```text
Inicial:   P=4 × I=3 = 12 (Alto)
Medida:    migrar a banda licenciada de 23 GHz
Residual:  P=1 × I=3 = 3 (Bajo)
```

## Ejemplos

> [!example] FTTH GPON (ISP regional)
> Permisos de postes atrasan el tendido. **P = 4:** 5 de 8 despliegues previos se atrasaron y la tramitación real toma 60-90 días frente a 30 planificados. **I = 4:** 6-8 semanas en la ruta crítica y 1.200 clientes en preventa. **R = 16 (Alto).** Mitigación: iniciar trámites en Preparar, priorizar sectores con factibilidad aprobada y evaluar canalización subterránea.

> [!example] Radioenlace rural
> Interferencia en 5,8 GHz. **P = 4:** 7 redes en el canal con piso de ruido de −78 dBm y torre compartida. **I = 3:** 450 clientes con servicio degradado, sin caída total. **R = 12 (Alto).** Residual tras migrar de banda: 3.

> [!example] Falla de OLT
> **P = 1:** AFR de 1,2 % anual, equipo nuevo con burn-in. **I = 5:** 3.200 clientes en una sola OLT y 15 días de reposición. **R = 5 (Bajo)**, pero es un **punto único de falla**: conviene mitigarlo igual con tarjeta de respaldo o equipo en stock.

## Plantilla de registro de riesgos

| ID | Causa → evento → efecto | Categoría | Fase | P | I | R | Nivel | Estrategia | Responsable | R residual |
|---|---|---|---|---|---|---|---|---|---|---|
| R01 | Permisos de postes → atraso del tendido → servicio tardío | Obra civil | Preparar | 4 | 4 | 16 | Alto | Mitigar | Jefe de proyecto | 8 |
| R02 | Espectro saturado → interferencia → degradación | Radioeléctrico | Diseñar | 4 | 3 | 12 | Alto | Evitar | Ingeniería RF | 3 |
| R03 | Falla de OLT → caída → clientes sin servicio | Hardware | Diseñar | 1 | 5 | 5 | Bajo | Mitigar | Ingeniería de red | 2 |

## Checklist

- [ ] Cada riesgo redactado como causa → evento → efecto
- [ ] Categoría y fase PPDIOO asignadas
- [ ] P con historial, mediciones y fichas técnicas
- [ ] I con atraso, sobrecosto, clientes y consecuencias contractuales
- [ ] Cálculo visible, clasificación, estrategia y responsable
- [ ] Riesgo residual recalculado y fuentes citadas

---
🔗 Relacionado: [[Cálculo de riesgos (Probabilidad × Impacto)]] · [[Capacity Planning y QoS]] · [[PPPoE para FTTH (MikroTik)]] · [[Plan de Respuesta a Incidentes (PRI)]]
