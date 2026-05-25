---
tags:
  - monitoreo
  - wireshark
  - analisis-de-trafico
  - ciberseguridad
aliases:
  - Análisis de tráfico
  - Captura de paquetes
---

# Wireshark

**Analizador de protocolos de red que captura y examina el tráfico en tiempo real. Sirve para diagnosticar problemas de red, optimizar el rendimiento y para tareas de ciberseguridad (detección de ataques, análisis forense, tráfico sospechoso).**

## Para qué sirve

- **Administración de redes**: diagnosticar fallas, verificar configuraciones, optimizar rendimiento.
- **Ciberseguridad**: detectar y analizar intentos de ataque, monitorear tráfico sospechoso, análisis forense de incidentes.

## Conceptos clave

1. **Captura de paquetes**: toma todos los paquetes que pasan por la interfaz/red a la que está conectado.
2. **Interfaz de usuario**: visualiza y filtra los paquetes capturados.
3. **Decodificación de protocolos**: interpreta y muestra los datos de cada protocolo.
4. **Filtros de captura y de visualización**: enfocan el análisis en datos específicos.
5. **Análisis y estadísticas**: herramientas integradas para interpretar y resumir.

## Flujo de trabajo

1. **Capturar tráfico** en la interfaz conectada a la red.
2. **Visualizar paquetes** en la interfaz gráfica.
3. **Aplicar filtros** para enfocar el análisis.
4. **Analizar datos** (encabezados y contenido de cada paquete).
5. **Guardar/Exportar** la captura para análisis posterior.

> [!info] Qué conviene dominar
> TCP/IP, UDP, HTTP, DNS y similares; uso de filtros de captura/visualización; lectura e interpretación de los datos; y técnicas de diagnóstico.

## Filtros de visualización útiles — Redes

```text
ip.addr == 192.168.1.1                       # IP específica
tcp                                          # solo TCP
tcp.port >= 20 && tcp.port <= 25             # rango de puertos
eth.addr == 00:0a:95:9d:68:16               # dirección MAC
http.request.method == "GET"                 # peticiones HTTP GET
dns                                          # consultas DNS
tls                                          # tráfico cifrado TLS/HTTPS (antes "ssl")
arp                                          # tráfico ARP
icmp                                         # tráfico ICMP
```

## Filtros de visualización útiles — Seguridad

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0     # escaneo SYN (estilo Nmap)
tcp.flags == 0x02                            # posible fuerza bruta (solo SYN)
tcp.port == 22                               # tráfico SSH
tcp.port == 23                               # Telnet (credenciales en claro)
ftp                                          # tráfico FTP
http contains "Authorization: Basic"         # credenciales HTTP básicas
smtp                                         # correo saliente
ip.flags.mf == 1                             # paquetes fragmentados
tcp.flags.syn == 1 && tcp.flags.ack == 0 && frame.time_delta < 0.0001   # posible DDoS/SYN flood
```

> [!warning] Telnet y HTTP básico
> Protocolos como Telnet, FTP o HTTP con autenticación básica transmiten credenciales en texto claro: con Wireshark son legibles. Úsalos solo en laboratorio y prefiere SSH/HTTPS en producción.

> [!tip] Captura vs. visualización
> Los **filtros de captura** descartan tráfico antes de almacenarlo (reducen el volumen). Los **filtros de visualización** se aplican sobre lo ya capturado sin perder datos. Los ejemplos de arriba son de visualización.

---
📎 Guías: ![[Wireshark-Guia.pdf]]

🔗 Relacionado: [[iperf3 test de velocidad real]] · [[Zabbix y SNMP en Cisco]] · [[Splunk Enterprise]]
