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
---

# iperf3: test de velocidad real

**Herramienta de código abierto para medir el rendimiento real de una red. Evalúa la velocidad de transferencia entre dos sistemas, junto con el jitter y la pérdida de paquetes, lo que ayuda a diagnosticar problemas, optimizar configuraciones y validar el ancho de banda contratado.**

## Modelo cliente-servidor

iperf3 funciona con dos extremos: uno actúa como **servidor** (escucha) y el otro como **cliente** (genera el tráfico de prueba). El resultado incluye **tasa de transferencia (throughput)**, **jitter** y **pérdida de paquetes**.

> [!info] Por qué "velocidad real"
> A diferencia de un speedtest contra internet, iperf3 mide el rendimiento entre dos puntos que tú controlas (LAN, enlace WAN, túnel), aislando el desempeño del enlace de la calidad del servidor remoto.

## Configuración del servidor

1. Descarga iperf3 desde el sitio oficial para tu sistema operativo.
2. Descomprime y abre una consola (CMD/terminal) en esa carpeta.
3. Pon el equipo en modo servidor escuchando en un puerto:

```bash
iperf3 -s -p 5005
```

- `-s`: modo servidor.
- `-p`: puerto de escucha.

## Configuración del cliente

En el otro equipo, repite la descarga/descompresión y ejecuta:

```bash
iperf3 -c <ip_servidor> -p 5005
```

- `-c <ip_servidor>`: modo cliente apuntando a la IP del servidor.
- `-p`: debe coincidir con el puerto del servidor.

> [!warning] El puerto debe coincidir
> El `-p` del cliente tiene que ser el mismo que el del servidor. Si el servidor escucha en `5005`, el cliente debe usar `5005`; de lo contrario la conexión falla.

> [!tip] Pruebas adicionales
> `-u` mide UDP (útil para evaluar jitter y pérdida en VoIP/streaming), `-R` invierte el sentido (descarga), `-t <seg>` define la duración y `-P <n>` lanza varios flujos en paralelo.

---
📎 Guías: ![[iperf3-Test-Velocidad.pdf]]

🔗 Relacionado: [[Wireshark]] · [[Capacity Planning y QoS]]
