---
tags:
  - auditoria
  - respuesta-incidentes
  - nist
  - csirt
aliases:
  - PRI
  - Respuesta a incidentes
  - Incident Response
  - NIST SP 800-61
---

# Plan de Respuesta a Incidentes (PRI)

Toda organización sufre incidentes; la diferencia entre una molestia y una catástrofe está en **cómo se responde**. Un PRI define **qué hacer, quién lo hace y en qué orden**. Se basa en **NIST SP 800-61** e **ISO/IEC 27035**.

## Evento, incidente y respuesta

| Concepto | Definición | Ejemplo |
|---|---|---|
| Evento | Cualquier ocurrencia observable; casi todos son normales | Un login fallido |
| Incidente | Evento o conjunto de eventos que compromete confidencialidad, integridad o disponibilidad | 500 logins fallidos y uno exitoso desde otro país |
| Respuesta | Proceso organizado, con fases, responsables y plazos, documentado de principio a fin | Activar el PRI |

## Marcos de referencia

| NIST SP 800-61 Rev. 2 | ISO/IEC 27035 | Qué se hace |
|---|---|---|
| Preparación | Planificar y preparar | Políticas, equipo, herramientas |
| Detección y análisis | Detección y reporte + evaluación y decisión | Identificar, validar y clasificar |
| Contención, erradicación y recuperación | Respuestas | Aislar, eliminar, restaurar |
| Actividad post-incidente | Lecciones aprendidas | Informe, mejoras, cierre |

La **Rev. 3 (2025)** alinea la respuesta con las funciones del **CSF 2.0** (Gobernar, Identificar, Proteger, Detectar, Responder, Recuperar). Las fases de la Rev. 2 siguen siendo la base didáctica. Es un **ciclo**: lo aprendido mejora la preparación del siguiente incidente.

## 1. Preparación

| Rol (CSIRT interno) | Responsabilidad |
|---|---|
| Líder de incidentes | Coordina y decide (jefe de TI o CISO) |
| Analista técnico | Investiga, contiene y erradica |
| Comunicaciones | Informa a clientes, prensa y autoridad |
| Legal / cumplimiento | Evalúa obligaciones legales (DPO) |
| Dirección | Aprueba decisiones críticas |

Recursos: lista de contactos al día (internos, proveedores, CSIRT nacional), inventario de activos críticos, **respaldos probados y fuera de línea**, herramientas (SIEM, IDS/IPS, EDR, logs centralizados), formularios de registro y *playbooks* por tipo de incidente.

> [!tip] Respaldo que no se prueba no existe
> "Los respaldos se prueban cada mes; última restauración exitosa el día X" vale. "Tenemos respaldos" no.

## 2. Detección y análisis

| Fuente | Ejemplo |
|---|---|
| SIEM / IDS | Alerta de [[Suricata]] por tráfico hacia una IP maliciosa conocida |
| Logs | 300 logins fallidos en 10 minutos |
| Usuarios | "Mis archivos terminan en .locked" |
| Terceros | El CSIRT nacional avisa que hay credenciales filtradas |

Preguntas: ¿qué pasó?, ¿cuándo empezó?, ¿qué sistemas y datos afecta?, ¿sigue activo?, ¿hay datos personales comprometidos?

| Severidad | Criterio | Respuesta |
|---|---|---|
| Baja | Un equipo, sin datos sensibles | < 72 h |
| Media | Varios equipos o servicio degradado | < 24 h |
| Alta | Servicio crítico caído o datos personales expuestos | < 4 h |
| Crítica | Operación paralizada, datos masivos, obligación legal | Inmediata |

Los plazos son de ejemplo: cada organización los fija según sus servicios críticos. La lógica es la de [[Cálculo de riesgos (Probabilidad × Impacto)]] aplicada al incidente en curso.

## 3. Contención, erradicación y recuperación

> [!warning] Primero la evidencia
> Antes de contener, preserva evidencia: imagen de disco, captura de memoria y copia de logs. **Desconecta, no apagues.**

| Etapa | Objetivo | Acciones |
|---|---|---|
| Contención corta | Frenar la propagación | Aislar el equipo de la red |
| Contención larga | Estabilizar | Bloquear IP en el firewall, deshabilitar cuentas comprometidas |
| Erradicación | Eliminar la causa raíz | Quitar malware y persistencias, cambiar credenciales, parchar, buscar otros equipos infectados |
| Recuperación | Volver a operar seguro | Restaurar desde respaldo limpio, validar integridad con hash, monitoreo reforzado de al menos 30 días |

"Se reinstaló y listo" no es recuperación: si no sabes cómo entró el atacante, volverá a entrar.

## 4. Post-incidente

Reunión de lecciones aprendidas **dentro de dos semanas** del cierre: línea de tiempo, qué funcionó, qué falló o se demoró, qué controles faltaban y qué cambia en el PRI. El informe final incluye línea de tiempo, causa raíz, impacto real (sistemas, usuarios, datos, horas, costo), acciones y mejoras con responsable y fecha.

## Marco legal en Chile

| Norma | A quién aplica | Obligación |
|---|---|---|
| Ley 21.663 (Marco de Ciberseguridad) | Servicios esenciales y operadores de importancia vital | Reportar al CSIRT nacional (ANCI): alerta temprana en **3 h**, segundo informe en **72 h** (24 h para OIV de servicios esenciales), informe final en **15 días** |
| Ley 21.719 (Protección de Datos Personales) | Quien trate datos personales | Notificar a la Agencia de Protección de Datos y, según el caso, a los titulares, sin dilaciones indebidas |
| Normativa sectorial (CMF y otras) | Banca, cooperativas, seguros | Reporte según el regulador |

Verifica los plazos vigentes en la ley y sus reglamentos al redactar el PRI.

## Ejemplos

> [!example] Ransomware en una cooperativa
> 08:15 usuarios reportan archivos `.locked`; 08:20 el EDR alerta cifrado masivo; 08:30 nota de rescate confirmada. **Crítica:** servidor de archivos y base de 5.200 socios afectados. **Contención:** VLAN de cuarentena, cuenta de servicio deshabilitada, captura de memoria e imagen de disco. **Erradicación:** el acceso inicial fue RDP expuesto con clave débil; se cierra RDP público y se resetean credenciales. **Recuperación:** respaldo offline del día anterior y monitoreo de 30 días. **Reporte:** CSIRT nacional en 3 h y agencia de datos. **Lecciones:** VPN con MFA, sin RDP público, RPO de 4 h.

> [!example] Phishing en una microempresa
> Un empleado entregó su clave de correo. **Media:** una cuenta, sin datos sensibles. Cambio de clave y cierre de sesiones, eliminación de una regla de reenvío creada por el atacante, MFA en la cuenta. **Lecciones:** MFA obligatorio y capacitación (ver [[GoPhish]]).

## Plantilla de registro

```text
ID:                   INC-AAAA-NNN
Detección (fecha/hora):
Reportado por:
Sistemas afectados:
Tipo:                 [ ] Malware [ ] Phishing [ ] Fuga de datos [ ] DDoS [ ] Acceso no autorizado [ ] Otro
Severidad:            [ ] Baja [ ] Media [ ] Alta [ ] Crítica
Datos personales:     [ ] Sí [ ] No   → cantidad aprox.:
Reporte requerido:    [ ] CSIRT nacional [ ] Agencia de datos [ ] Regulador
Línea de tiempo:      HH:MM — acción — responsable
Contención:
Erradicación (causa raíz):
Recuperación:
Lecciones aprendidas:
Fecha de cierre:
```

## Checklist

- [ ] Equipo con roles y contactos, inventario de activos, respaldos probados, playbooks
- [ ] Fuentes de detección y tabla de severidad con plazos
- [ ] Procedimiento de preservación de evidencia y criterios para volver a operar
- [ ] Reunión de lecciones aprendidas y formato de informe
- [ ] Leyes aplicables y plazos de notificación documentados
- [ ] Simulacro al menos una vez al año

## Referencias

- NIST SP 800-61 Rev. 2 y Rev. 3
- ISO/IEC 27035-1:2023
- Ley 21.663 y Ley 21.719 (Chile)
- CSIRT de Gobierno de Chile: <https://www.csirt.gob.cl>

---
🔗 Relacionado: [[Cálculo de riesgos (Probabilidad × Impacto)]] · [[Marcos normativos (NIST SP 800-53, Well-Architected y CSA CCM)]] · [[Suricata]] · [[Splunk Enterprise]] · [[07 - Logs y auditoría]] · [[09 - Backups y recuperación]]
