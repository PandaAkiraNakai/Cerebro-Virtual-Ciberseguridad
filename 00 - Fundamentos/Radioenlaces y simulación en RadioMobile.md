---
tags:
  - fundamentos
  - inalambrico
  - radioenlaces
  - telecomunicaciones
aliases:
  - RadioMobile
  - Radio Mobile
  - Presupuesto de enlace
  - Link budget
  - Zona de Fresnel
---

# Radioenlaces y simulación en RadioMobile

**RadioMobile** simula enlaces de radio punto a punto sobre modelos digitales de terreno. Calcula pérdidas, señal recibida y margen para decidir si un enlace funcionará antes de subir a una torre.

## Ubicación y antenas

| Parámetro | Significado |
|---|---|
| Latitud / longitud | Posición de cada estación en grados decimales |
| Elevación del terreno | Altura sobre el nivel del mar; se obtiene del modelo de terreno |
| Altura de la antena | Del suelo al centro de radiación; se suma a la elevación |
| Azimut (TN / MG) | Dirección horizontal: norte verdadero o magnético |
| Inclinación (tilt) | Ángulo vertical: positivo hacia arriba, negativo hacia abajo |

Más altura significa mejor línea de vista: se esquivan árboles, casas y la curvatura de la Tierra. Ejemplo típico: 2 m en el techo de una caseta en un extremo y un poste de 10 m en el otro.

## Sistema de radio

| Parámetro | Ejemplo | Comentario |
|---|---|---|
| Frecuencia (MHz) | 900 / 5800 | Las bajas atraviesan mejor la vegetación; las altas dan más capacidad |
| Potencia TX | 30 dBm = 1 W | Limitada por la normativa de cada banda |
| Pérdida de línea TX / RX | 1-3 dB | Cables y conectores entre radio y antena: cortos y de calidad |
| Ganancia de antena TX / RX | dBi | Cuánto concentra la antena la energía en una dirección |
| Sensibilidad RX | −113 dBm | Señal mínima que el receptor descifra; más negativo es mejor |

## Condiciones de diseño

| Opción | Efecto |
|---|---|
| Fiabilidad requerida (%) | Tiempo que el enlace debe funcionar; 95 % acepta hasta un 5 % de falla al año |
| Usar cobertura del terreno | Considera cerros y valles que pueden bloquear la señal |
| Usar dos rayos | Suma la reflexión en el suelo, que puede reforzar o cancelar la señal directa |

## Pérdidas de propagación

| Pérdida | Causa |
|---|---|
| Espacio libre (FSPL) | Distancia y frecuencia, sin obstáculos |
| Obstrucción | Cerros o edificios en el trayecto |
| Bosque / urbanización | Vegetación densa o infraestructura urbana |
| Estadística | Margen por clima, reflexiones e interferencia |
| Total | Suma de todas |

## Presupuesto de enlace

```text
FSPL (dB)        = 32,44 + 20·log10(d en km) + 20·log10(f en MHz)
PIRE (dBm)       = Ptx − pérdida de línea TX + ganancia de antena TX
Señal Rx (dBm)   = PIRE − pérdidas de propagación + ganancia antena RX − pérdida de línea RX
Margen (dB)      = Señal Rx − sensibilidad Rx
```

> [!example] 10 km a 5,8 GHz
> FSPL = 32,44 + 20 + 75,3 ≈ 127,7 dB. Con 20 dBm de potencia, antenas de 23 dBi y 1 dB de cable en cada extremo: Rx ≈ 20 − 1 + 23 − 127,7 + 23 − 1 = −63,7 dBm. Con sensibilidad de −80 dBm, el margen es de unos 16 dB.

**Margen de desvanecimiento:** más de 20 dB es lo ideal; con menos, la lluvia, el viento o la interferencia pueden cortar el enlace.

## Zona de Fresnel

La línea de vista visual no basta: la primera **zona de Fresnel**, un elipsoide alrededor del rayo directo, debe quedar despejada al menos en un **60 %**.

```text
r (m) = 17,32 · √( d1 · d2 / (f · d) )      d1, d2, d en km · f en GHz
```

En el punto medio de un enlace de 10 km a 5,8 GHz, el radio es de unos 11,4 m.

## Resultados que entrega RadioMobile

| Resultado | Significado |
|---|---|
| Distancia (km) | Largo del enlace según las coordenadas |
| Precisión (m) | Resolución del terreno usado; menor es más preciso |
| PIRE / EIRP (W) | Potencia efectiva irradiada |
| Ganancia del sistema (dB) | Ganancias menos pérdidas fijas |
| Señal recibida (dBm y µV) | Nivel que llega al receptor |
| Margen (dB) | Señal recibida menos sensibilidad |
| Peor despeje de Fresnel | Si es menor a 0,6F1, sube las antenas o cambia la ruta |

---
🔗 Relacionado: [[Capacity Planning y QoS]] · [[Análisis de riesgos en proyectos de telecomunicaciones]] · [[PPPoE para FTTH (MikroTik)]] · [[Meshtastic comandos y scripts]]
