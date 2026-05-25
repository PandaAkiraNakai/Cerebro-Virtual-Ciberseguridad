---
tags:
  - ids
  - snort
  - pfsense
aliases:
  - Snort
---

# Snort

**Snort** — herramienta de código abierto para monitorear el tráfico de red y detectar amenazas. Funciona como **IDS** (Sistema de Detección de Intrusiones) y como **IPS** (Sistema de Prevención de Intrusiones), ofreciendo protección en tiempo real.

## ¿Para qué sirve?

- **Detección de intrusiones (IDS):** monitorea el tráfico buscando patrones sospechosos según reglas predefinidas y alerta a los administradores.
- **Prevención de intrusiones (IPS):** además de detectar, bloquea el tráfico malicioso de forma proactiva en tiempo real.
- **Registro de tráfico:** guarda el tráfico para auditoría, análisis forense y revisión de incidentes.

## Modos de operación

| Modo | Propósito |
|---|---|
| Sniffer | Muestra los paquetes que pasan por la red en tiempo real, sin aplicar reglas. |
| Logger | Guarda los paquetes capturados en un archivo para análisis posterior. |
| IDS/IPS | Detecta o bloquea el tráfico según las reglas configuradas. Es el modo principal. |

## Componentes clave

- **Reglas:** definen cómo se comporta Snort ante tráfico sospechoso (alertar, registrar o bloquear). Las *opciones de las reglas* permiten condiciones detalladas como contenido, flags TCP y otros parámetros.
- **Preprocesadores:** analizan y preparan el tráfico antes de aplicar las reglas (fragmentación IP, reensamblaje TCP, inspección HTTP). Mejoran la detección.
- **Alertas y logs:** generan notificaciones en tiempo real y registran los paquetes que cumplen reglas.
- **Interfaces de salida:** integran Snort con plataformas externas como Snorby, BASE o Kibana para visualizar las detecciones.

## Conceptos importantes

> [!info] Falsos positivos y negativos
> Ajustar reglas y preprocesadores es fundamental para minimizar errores: un **falso positivo** alerta sobre tráfico legítimo; un **falso negativo** deja pasar un ataque real.

> [!tip] Actualización de firmas
> Las firmas de ataque deben actualizarse con frecuencia para detectar nuevas vulnerabilidades.

## Instalación en pfSense

1. Ve a **System > Package Manager** y abre la pestaña **Available Packages**.
2. Busca **Snort** en la lista y haz clic en **Install**.
3. Espera a que termine la instalación para comenzar a configurarlo.

## Configuración en pfSense

Tras instalar, configura la(s) lista(s) de reglas, marca las casillas necesarias, guarda los cambios y **actualiza la lista de reglas**.

### Activar el modo IPS

1. Ve a **Services > Snort**, pestaña **Global Settings** y marca **Enable Block Offenders**.
2. En la pestaña **Interfaces**, selecciona la interfaz (WAN o LAN) y habilita **Block Offenders** para esa interfaz.
3. Guarda los cambios. Snort bloqueará automáticamente el tráfico malicioso en las interfaces configuradas.

### Categorías de tráfico (WAN/LAN Category)

Permite especificar qué tráfico inspecciona Snort en cada interfaz. Puedes analizar tráfico entrante (*Inbound*), saliente (*Outbound*) o ambos (*Both*) para personalizar el monitoreo.

### Preprocesadores (LAN/WAN Preprocessors)

Activan módulos que procesan el tráfico antes del análisis:

| Preprocesador | Función |
|---|---|
| Frag3 | Reensambla paquetes fragmentados |
| Stream5 | Rastrea y sigue conexiones |

Habilitarlos en las interfaces LAN o WAN mejora la precisión y eficiencia de la detección.

---
📎 Guías: ![[Snort-Guia.pdf]]

🔗 Relacionado: [[Suricata]] · [[Portal cautivo en pfSense]] · [[iptables]]
