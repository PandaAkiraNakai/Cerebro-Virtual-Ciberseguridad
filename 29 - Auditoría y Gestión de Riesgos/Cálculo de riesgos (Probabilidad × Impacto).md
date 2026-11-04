---
tags:
  - auditoria
  - riesgos
  - sgsi
aliases:
  - Cálculo de riesgos
  - Matriz de riesgos
  - Riesgo = P × I
---

# Cálculo de riesgos (Probabilidad × Impacto)

Para priorizar amenazas hay que **cuantificarlas**. La fórmula base combina dos dimensiones en una escala de 1 a 5:

```text
Riesgo (R) = Probabilidad (P) × Impacto (I)        rango 1 a 25
```

| Dimensión | Pregunta | Depende de |
|---|---|---|
| Probabilidad | ¿Qué tan probable es que ocurra? | Vulnerabilidades, intentos previos, capacidad del atacante |
| Impacto | ¿Qué tan grave sería? | Datos afectados, operación perdida, daño legal y reputacional |

## Los valores salen de datos, no de opiniones

```text
✗ "P = 4" porque creo que es probable
✓ "P = 4" porque hay datos objetivos que muestran probabilidad alta
```

| Dato | Dónde buscarlo |
|---|---|
| Ataques históricos en el sector | Reportes públicos, CSIRT nacionales, INCIBE, Have I Been Pwned |
| Intentos contra tu organización | Logs de firewall, [[Suricata]] u otro IDS, SIEM |
| Vulnerabilidades | Auditoría técnica, pentesting, escaneo |
| Contexto | Usuarios reales, datos que se tratan, dependencias críticas |
| Regulación | Leyes de protección de datos y normas sectoriales |

Busca **patrones sostenidos**, no eventos aislados: "pasó hace dos años" no es una tendencia. Si no tienes datos, no asignes números: investiga primero.

## Escalas de ejemplo

> [!warning] Ajústalas a tu sector
> Los umbrales son referenciales; cada organización los valida con sus propios datos y los documenta en su política de riesgos o SGSI.

| P | Nivel | Evidencia típica |
|---|---|---|
| 1 | Muy baja | Menos de 1 ataque al año en el sector y sin vulnerabilidades |
| 2 | Baja | 1-2 al año y vulnerabilidades menores |
| 3 | Media | 3-5 al año y vulnerabilidades conocidas |
| 4 | Alta | 6-10 al año e intentos documentados contra la organización |
| 5 | Muy alta | Más de 10 al año o intentos activos ahora |

| I | Nivel | Evidencia típica |
|---|---|---|
| 1 | Muy bajo | Menos de 50 usuarios, datos no clasificados |
| 2 | Bajo | 50-500 usuarios, datos internos no regulados |
| 3 | Medio | 500-5.000 usuarios, datos personales |
| 4 | Alto | 5.000-50.000 usuarios, datos financieros o sistemas críticos caídos |
| 5 | Muy alto | Más de 50.000 usuarios, operación paralizada o infracción legal segura |

## Procedimiento

1. **Define la amenaza** concreta: fuga de datos de clientes, DDoS, acceso no autorizado a la base, malware.
2. **Estima P** con ataques del sector, intentos contra ti y hallazgos de auditoría.
3. **Estima I** con el número exacto de usuarios, el tipo de datos, la ley aplicable y la criticidad del sistema.
4. **Multiplica** y deja el cálculo visible.
5. **Clasifica:**

| Riesgo | Rango | Acción |
|---|---|---|
| Bajo | 1-5 | Monitorear |
| Medio | 6-9 | Mejorar controles |
| Alto | 10-16 | Intervenir pronto |
| Crítico | 17-25 | Intervenir ya |

## Matriz

```text
            IMPACTO
         1   2   3   4   5
P   1    1   2   3   4   5
R   2    2   4   6   8  10
O   3    3   6   9  12  15     ← P=3, I=5 → 15 (Alto)
B   4    4   8  12  16  20     ← P=4, I=5 → 20 (Crítico)
    5    5  10  15  20  25
```

## Ejemplos

> [!example] Fuga de base de datos (empresa de servicios)
> **P = 4:** base MySQL expuesta en el 3306 sin WAF, intentos de inyección SQL cada 2-3 días en los logs, brechas similares en el sector y sin MFA.
> **I = 5:** casi 9.000 clientes con identificador nacional, correo y transacciones; ley de datos personales aplicable; el 95 % de los ingresos depende del sistema.
> **R = 4 × 5 = 20 → Crítico.**

> [!example] DDoS (microempresa)
> **P = 2:** cero intentos en 12 meses de logs y ataques raros en el sector.
> **I = 3:** unas 200 personas afectadas, datos internos no regulados, se puede operar sin sistema de 4 a 8 horas.
> **R = 2 × 3 = 6 → Medio.**

## Checklist

- [ ] Amenaza definida con precisión
- [ ] P respaldada con datos del sector, logs propios y auditoría
- [ ] I respaldada con inventario real, tipos de datos, ley aplicable y criticidad
- [ ] Cálculo visible y clasificación según la tabla
- [ ] Criterios tomados de la política de la organización
- [ ] Fuentes citadas

---
🔗 Relacionado: [[Análisis de riesgos en proyectos de telecomunicaciones]] · [[Marcos normativos (NIST SP 800-53, Well-Architected y CSA CCM)]] · [[Plan de Respuesta a Incidentes (PRI)]]
