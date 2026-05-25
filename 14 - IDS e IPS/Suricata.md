---
tags:
  - ids
  - suricata
  - linux
aliases:
  - Suricata
---

# Suricata

**Suricata** — sistema de detección y prevención de intrusiones (**IDS/IPS**) de código abierto. Monitorea el tráfico de red en tiempo real, lo analiza buscando patrones sospechosos y puede bloquear actividades maliciosas.

## Cómo funciona

1. **Captura de tráfico:** "escucha" el tráfico que pasa por una interfaz específica.
2. **Decodificación:** descompone los paquetes en elementos comprensibles (IP, puertos, protocolos).
3. **Inspección profunda (DPI):** analiza el contenido para identificar aplicaciones, comandos y datos maliciosos.
4. **Comparación con reglas:** contrasta el tráfico con reglas que describen patrones maliciosos.
5. **Generación de alertas:** si coincide con una regla, emite una alerta detallada.
6. **Registro:** guarda todos los eventos para análisis posterior.
7. **Prevención (opcional):** configurado como IPS, bloquea el tráfico malicioso en tiempo real.

> [!info] Reglas: el núcleo de Suricata
> Las reglas definen qué tráfico es sospechoso. Suricata viene preconfigurado para usar las reglas **Emerging Threats Open (ET Open)**, pero hay que descargarlas.

## Instalación

```bash
# Preparar el sistema
sudo apt update
sudo apt upgrade

# Debian y Kali Linux
sudo apt install suricata

# Ubuntu (repositorio oficial)
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata
```

Descargar las reglas ET Open (esenciales para funcionar):

```bash
sudo suricata-update
```

## Configuración (suricata.yaml)

Edita `/etc/suricata/suricata.yaml`:

| Buscar | Cambio |
|---|---|
| `HOME_NET` | Reemplaza la IP por la red local a proteger |
| `af-packet:` | Ajusta la interfaz de captura de tu equipo |
| `rule-files:` | Agrega la ruta de tus reglas personalizadas |
| `pcap:` | Ajusta la interfaz (método pcap) |
| `community-id:` | Cambia a `True` para mejor correlación en SIEM |

## Validación y arranque

```bash
suricata -T -c <config>                    # Valida la configuración
suricata -c <config> -i <interfaz>         # Modo IDS en una interfaz
suricata -t                                # Modo de prueba (análisis de capturas)
suricata -c <config> -s <reglas> -i <interfaz>  # Con reglas personalizadas
```

- `<config>`: ruta a `suricata.yaml`
- `<reglas>`: ruta al archivo de reglas personalizadas
- `<interfaz>`: nombre de la interfaz a monitorear

```bash
# Iniciar
suricata -c /etc/suricata/suricata.yaml -i eth0

# Seguir alertas en vivo
tail -f /var/log/suricata/fast.log
```

## Administración del servicio

```bash
sudo systemctl start suricata     # Iniciar
sudo systemctl stop suricata      # Detener
sudo systemctl restart suricata   # Reiniciar
sudo systemctl status suricata    # Estado
sudo systemctl enable suricata    # Inicio automático al arrancar
sudo systemctl disable suricata   # Deshabilitar inicio automático
```

## Pruebas

- **Ataques simulados:** usa Nmap o Metasploit para verificar la detección.
- **Tráfico malicioso (con precaución):** genera tráfico que coincida con tus reglas.
- **Análisis de registros:** revisa `/var/log/suricata/` para confirmar que los eventos se registran.

## Archivos de registro

| Archivo | Contenido |
|---|---|
| `eve.json` | Núcleo del registro: alertas, flujos de red, DNS, HTTP/TLS. Clave para análisis forense. |
| `fast.log` | Resumen ejecutivo de alertas: marca de tiempo, origen/destino, regla activada, clasificación y prioridad. |
| `stats.log` | Rendimiento: uso de CPU/memoria, paquetes procesados, reglas coincidentes. |
| `suricata.log` | Diario del sistema: mensajes de inicio, errores, advertencias y notificaciones. |

## Solución de errores y rendimiento

> [!warning] Error "fanout not supported by kernel"
> Indica que la función *fanout* de AF_PACKET (distribuye el procesamiento entre núcleos de CPU) no puede usarse.
>
> - **Kernel incompatible:** actualiza el kernel a una versión compatible.
> - **Cluster ID en uso:** cambia `cluster-id` en `suricata.yaml` (p. ej. de 99 a 100) y verifica procesos con `ps aux | grep suricata`.
> - **Cluster type incorrecto:** prueba otro `cluster_type` y ajusta la afinidad de CPU.

### Modos de cluster

| Modo | Descripción |
|---|---|
| `cluster_flow` (recomendado) | Agrupa paquetes por flujo (misma IP/puerto origen-destino). Mejor localidad de datos. Ideal para la mayoría de sistemas. |
| `cluster_cpu` | Asigna paquetes secuencialmente a núcleos. Útil si la NIC no soporta RSS. Requiere ajuste de afinidad. |
| `cluster_qm` | Como `cluster_cpu`, con colas MPMC para mejor eficiencia. Requiere ajuste de afinidad. |

## Reglas de terceros

| Fuente | Descripción |
|---|---|
| Emerging Threats Open (ET Open) | Catálogo gratuito de alta calidad. Pilar de la comunidad. |
| Proofpoint ET Pro | Opción comercial: reglas adicionales, soporte prioritario, inteligencia exclusiva. |
| Otras | Repositorios especializados por industria y proveedores comerciales. |

Descarga e instalación de ET Open:

```bash
wget https://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
tar -xzf emerging.rules.tar.gz -C /etc/suricata/rules
nano /etc/suricata/rules/suricata.yaml   # Descomentar líneas relevantes (rule-files)
systemctl restart suricata
```

> [!tip] Mantenimiento de reglas
> Automatiza la actualización con herramientas como Oinkmaster o pulledpork, evalúa el impacto en el rendimiento y ajusta las reglas para minimizar falsos positivos.

---
📎 Guías: ![[Suricata-Guia.pdf]]

🔗 Relacionado: [[Snort]] · [[iptables]] · [[Portal cautivo en pfSense]]
