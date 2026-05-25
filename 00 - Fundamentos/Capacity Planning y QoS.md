---
tags:
  - fundamentos
  - qos
  - diseño-de-red
aliases:
  - Cálculo de capacidad de red
  - Capacity Planning
---

# Cálculo de Capacidad de Red con Overhead y QoS

Al dimensionar el ancho de banda de una red **no basta con sumar el consumo de las aplicaciones**. Hay que contemplar **overhead** y **reserva para Calidad de Servicio ([[QoS para voz sin switch administrable|QoS]])**.

## 1. Conceptos

- **Overhead (≈ 30 %)**: cabeceras de protocolos (Ethernet, IP, TCP/UDP), retransmisiones y variaciones de tráfico. No existe una norma fija; es un **criterio de diseño** recomendado por fabricantes (Cisco, Juniper, ITU-T).
- **QoS (reserva extra)**: un **10–15 % adicional** para asegurar rendimiento en aplicaciones críticas como **voz y video en tiempo real**.

## 2. Ejemplo de cálculo

**Escenario:**
- 20 usuarios con datos (≈ 1 Mbps c/u)
- 5 usuarios en videollamadas HD (≈ 2 Mbps c/u)
- 2 líneas VoIP (≈ 0.1 Mbps c/u)

| Paso | Cálculo | Resultado |
|---|---|---|
| 1. Tráfico bruto | 20×1 + 5×2 + 2×0.1 | **30.2 Mbps** |
| 2. + Overhead 30 % | 30.2 × 1.3 | 39.26 Mbps |
| 3. + Reserva QoS 10 % | 39.26 × 1.1 | 43.18 Mbps |

> [!success] Ancho de banda recomendado ≈ **45 Mbps**

## 3. Codecs de Voz

| Codec  | Bitrate (Kbps) | Comentario |
|--------|----------------|------------|
| G.711  | 64             | Calidad alta, consumo alto |
| G.729  | 8              | Calidad aceptable, bajo BW |
| Opus   | 6–510          | Flexible, usado en WebRTC |
| AMR-WB | 12.65–23.85    | Eficiente, VoLTE |

## 4. Codecs de Video

| Codec | Bitrate típico (Mbps) | Comentario |
|-------|-----------------------|------------|
| H.264 | 1–4 (SD/HD)           | Amplio soporte, estándar |
| H.265 | 0.5–2 (SD/HD)         | Más eficiente, menor bitrate |
| VP8   | 1–2 (HD)              | Usado en WebRTC |
| VP9   | 0.5–1.5 (HD)          | Mejor compresión que VP8 |
| AV1   | 0.3–1 (HD)            | Muy eficiente, más reciente |

## 5. Métricas clave de QoS

- **Throughput**: tasa efectiva de transmisión.
- **Latencia**: tiempo de tránsito de un paquete.
- **Jitter**: variación de la latencia.
- **Packet loss**: pérdida de paquetes, crítico en voz/video.

---
🔗 Relacionado: [[QoS para voz sin switch administrable]] · [[QoS en router y switch con FreePBX]]
