---
tags:
  - monitoreo
  - rendimiento
  - iperf3
  - throughput
aliases:
  - iperf3
  - iperf3: test de velocidad real
  - Test de velocidad de red
  - Throughput
---

# iperf3: test de velocidad real

**iperf3** genera tráfico controlado entre un **cliente** y un **servidor** para medir el rendimiento de un camino IP: throughput, retransmisiones TCP, jitter y pérdida UDP. A diferencia de un speedtest contra Internet, mide entre dos puntos que tú controlas (LAN, enlace WAN, túnel).

```text
CLIENTE ─────── tráfico ───────► SERVIDOR      (con -R se invierte)
```

> [!warning] iperf3 no certifica cables
> Mide el camino completo: cable, conectores, NIC, driver, CPU, sistema operativo, TCP/UDP y la propia aplicación. Puede revelar que un enlace rinde menos de lo esperado, pero no demuestra que un cable cumple Cat 5e o Cat 6. Para eso existen certificadores de cableado.

## Conceptos

| Término | Significado |
|---|---|
| Ancho de banda nominal | Capacidad teórica o contratada |
| Throughput | Tasa efectiva observada en la prueba |
| Goodput | Datos útiles entregados, sin overhead ni retransmisiones |
| Latencia / RTT | Tiempo de tránsito / ida y vuelta |
| Jitter | Variación del tiempo entre paquetes; clave en VoIP y video |
| Pérdida | Datagramas UDP que no llegan |
| Retransmisiones | Segmentos TCP reenviados |
| MTU / MSS | Tamaño máximo del paquete / del segmento TCP |
| Stream | Flujo independiente generado por iperf3 |

## Instalación y uso básico

```bash
sudo apt install iperf3        # Debian, Ubuntu, Kali
iperf3 --version

iperf3 -s                      # servidor (puerto 5201 por defecto)
iperf3 -s -p 5005              # servidor en otro puerto
iperf3 -c 192.168.100.1        # cliente
iperf3 -c 192.168.100.1 -p 5005
```

En Windows se descarga el binario y se ejecuta desde una consola en esa carpeta. Abre el puerto en el firewall del servidor (`sudo ufw allow 5201`).

## Pruebas

| Prueba | Comando | Para qué |
|---|---|---|
| TCP base | `iperf3 -c IP -t 30 -i 2` | Throughput promedio y retransmisiones, con reporte cada 2 s |
| Varios streams | `iperf3 -c IP -P 4 -t 30` | Ver si un solo flujo no alcanzaba a llenar el enlace |
| Inversa | `iperf3 -c IP -R -t 30` | Comparar subida y bajada |
| Bidireccional | `iperf3 -c IP --bidir -t 30` | Full-duplex, asimetrías y CPU bajo carga simultánea |
| UDP | `iperf3 -c IP -u -b 100M -t 20` | Jitter y pérdida a una tasa objetivo |
| Omitir arranque | `iperf3 -c IP -t 30 -O 5` | Excluye los primeros 5 s (slow start de TCP) |
| JSON | `iperf3 -c IP -J > resultado.json` | Automatización y análisis con scripts |
| A archivo | `iperf3 -c IP --logfile resultado.txt` | Evidencia para el informe |

> [!info] `-b` no es el resultado
> En UDP, `-b` fija la tasa que se **intenta** generar. Si el camino no la sostiene, aparece pérdida. El throughput real se lee en la salida.

### Opciones avanzadas

| Opción | Efecto |
|---|---|
| `-l 64K` | Largo del buffer de lectura/escritura |
| `-w 1M` | Ventana o buffer del socket TCP |
| `-N` | TCP_NODELAY: desactiva Nagle |
| `-M 1400` | MSS de TCP |
| `-4` / `-6` | Forzar IPv4 o IPv6 |
| `-n 1G` / `-k 100000` | Terminar por bytes o por bloques en vez de tiempo |
| `-B IP` / `--bind-dev eth0` | Elegir la IP o interfaz de origen |
| `-C cubic` | Algoritmo de congestión TCP (Linux y FreeBSD) |
| `-A 2` | Afinidad de CPU: ¿la CPU limita? |
| `-Z` | Zero-copy: menos copias al enviar (no lo uses en la línea base) |
| `-F archivo` | Leer o escribir desde disco: ¿el almacenamiento limita? (solo TCP) |
| `-S` / `--dscp` | Marcar el tráfico para probar QoS (ver [[Capacity Planning y QoS]]) |
| `--mptcp` | Multipath TCP, si el sistema lo soporta |

La disponibilidad depende de la versión y la compilación: revisa `iperf3 --help`.

## Interpretar resultados

```text
[SUM]  0.00-30.00 sec  3.27 GBytes  936 Mbits/sec
```

- **Throughput:** "se observó un throughput promedio de unos 936 Mbit/s en esta prueba", no "el cable tiene 936 Mbit/s". En Gigabit Ethernet, el máximo práctico de TCP ronda 940 Mbit/s por el overhead de los encabezados.
- **Retransmisiones:** indican pérdida, congestión o un receptor que no da abasto. Por sí solas no prueban que el cable esté malo.
- **Jitter:** relevante para VoIP, videoconferencia, streaming y tiempo real.
- **Pérdida UDP:** aparece cuando la tasa generada supera lo que el camino sostiene. No hay un umbral universal como "menos de 0,1 %": depende del medio, la carga, los buffers y el objetivo de la prueba.

### Dónde está el cuello de botella

Cable → conectores → NIC → driver → CPU → sistema operativo y buffers → TCP/UDP → iperf3. Revisa primero lo que negoció el enlace:

```bash
sudo ethtool eth0        # Speed, Duplex, Link detected
```

## Metodología

- **Mantén constantes** equipos, NIC, cable, versión de iperf3, IP, duración, dirección y protocolo, sin tráfico externo relevante.
- **Cambia una sola variable por vez** (por ejemplo, `-P 1`, `-P 2`, `-P 4`, `-P 8`).
- **Repite** cada prueba al menos tres veces y promedia.

| Error frecuente | Por qué es incorrecto |
|---|---|
| "Si el cable es Cat 6, debe dar 10 Gbit/s" | La categoría no fija la velocidad negociada; mandan las NIC |
| "iperf3 certificó el cable" | Mide rendimiento IP, no parámetros físicos |
| "Hay pérdida UDP, el cable está malo" | Puede ser saturación, buffers, CPU o configuración |
| "El `-b` es lo que obtuve" | Es la tasa objetivo |
| "Con una prueba basta" | No permite evaluar la repetibilidad |

## Laboratorio: evaluar un enlace con un cable

```text
Equipo A 192.168.100.1/24 ──── cable bajo prueba ──── Equipo B 192.168.100.2/24
```

1. Registra la categoría rotulada, el largo, los conectores, las NIC, el sistema operativo y la versión de iperf3.
2. Verifica la negociación con `ethtool` y la conectividad con `ping`.
3. Servidor en A: `iperf3 -s`.
4. Desde B: TCP base, `-P 4`, `-R` y `--bidir`, cada una 30 s con `-i 2` y tres repeticiones.
5. UDP con carga creciente: `-u -b 100M`, `500M`, `800M`, `900M`. Anota throughput, jitter, pérdida y datagramas.

| Prueba | Rep. 1 | Rep. 2 | Rep. 3 | Promedio |
|---|---|---|---|---|
| TCP | | | | |
| TCP `-P 4` | | | | |
| Inversa | | | | |
| Bidireccional | | | | |

> [!example] Lectura de resultados
> TCP con un flujo 915 Mbit/s, con cuatro 942 Mbit/s, inversa 939 Mbit/s y bidireccional 930 + 928 Mbit/s. El enlace rinde cerca del máximo práctico de Gigabit. Los cuatro flujos subieron el resultado, así que un solo flujo no saturaba el camino. Funciona en full-duplex. Nada de esto certifica físicamente el cable.

Desafío: con enlace negociado a 1 Gbit/s, un flujo TCP da 420 Mbit/s, cuatro flujos 790 Mbit/s y UDP 850 Mbit/s con 0,8 % de pérdida. ¿Problema físico, CPU, buffers, límite de un solo flujo, saturación o configuración? Demuéstralo con pruebas nuevas.

**Informe:** identificación, topología, características del cable, configuración (IP, velocidad, dúplex, NIC), comandos usados, tablas de resultados, análisis y conclusión basada en la evidencia.

> [!warning] Solo en redes autorizadas
> No generes tráfico intenso, sobre todo UDP, en redes productivas o compartidas sin permiso. Define antes la duración y la tasa máxima.

Referencias: <https://software.es.net/iperf/> y <https://github.com/esnet/iperf>.

---
📎 Guías: ![[iperf3-Test-Velocidad.pdf]]

🔗 Relacionado: [[Wireshark]] · [[Capacity Planning y QoS]] · [[02 - Red y conectividad]] · [[Configuración de red por distribución]]
