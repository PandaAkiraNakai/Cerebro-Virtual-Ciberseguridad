---
tags:
  - iot
  - esp8266
  - arduino
  - thingsboard
aliases:
  - ESP8266 firmware
  - Sketches ESP8266
---

# ESP8266: sketches y firmware

**Colección de sketches Arduino para el ESP8266 (NodeMCU) orientados a IoT y pentesting WiFi: monitor de paquetes/deauth, lector RFID, sensores con envío a ThingsBoard por MQTT/HTTP y control de LED por web. Incluye firmware precompilado de descarga.**

> [!warning] Privacidad: los sketches originales traían SSID/contraseñas WiFi y *access tokens* de ThingsBoard reales. Aquí y en los archivos adjuntos se reemplazaron por placeholders (`TU_WIFI_SSID`, `TU_WIFI_PASSWORD`, `TU_TOKEN_THINGSBOARD`, `IP_DEL_SERVIDOR_THINGSBOARD`). Rellénalos con tus propios valores antes de compilar.

## Monitor de paquetes 2.4 GHz (detector de deauth)

Sketch basado en el proyecto *spacehuhn* (Stefan Kremser). Pone el ESP8266 en modo promiscuo, cuenta paquetes por canal y detecta tramas de **deauthentication** (subtipos `0xA0` / `0xC0`), mostrando un gráfico en una pantalla OLED SSD1306/SH1106. Cambia de canal con el botón FLASH y enciende un LED de alarma al superar el umbral de deauths.

Fragmento clave (sniffer y detección de deauth):

```cpp
void sniffer(uint8_t *buf, uint16_t len) {
  pkts++;
  if (buf[12] == 0xA0 || buf[12] == 0xC0) {  // tramas de deauth
    deauths++;
  }
}

// ...en setup():
wifi_set_opmode(STATION_MODE);
wifi_promiscuous_enable(0);
WiFi.disconnect();
wifi_set_promiscuous_rx_cb(sniffer);   // registra el callback del sniffer
wifi_set_channel(curChannel);
wifi_promiscuous_enable(1);            // activa modo promiscuo
```

El sketch completo (202 líneas) está adjunto.

## Lector RFID RC522 (herramienta de pentest)

ESP8266 + módulo **RC522**. Crea un Access Point y sirve un dashboard estilo "hacker" donde muestra las tarjetas detectadas, su tipo y una clasificación heurística de si son potencialmente *clonables* (p. ej. MIFARE Classic).

> [!info] La herramienta **solo identifica tipos de tarjeta potencialmente vulnerables**; NO realiza cracking ni clonado. El AP traía credenciales por defecto (sustituidas por `TU_AP_SSID` / `TU_AP_PASSWORD`). Úsala solo con fines educativos y autorizados.

El sketch completo (244 líneas) está adjunto.

## Sensor temperatura + humedad (DHT22 → ThingsBoard)

ESP8266 con sensor **DHT22 (AM2302)**. Publica telemetría a ThingsBoard por MQTT (`v1/devices/me/telemetry`) y expone un panel web con glassmorphism que se autorefresca cada 5 s.

```cpp
const char* mqttServer = "IP_DEL_SERVIDOR_THINGSBOARD";
const int mqttPort = 1883;
const char* token = "TU_TOKEN_THINGSBOARD";

void reconnectMQTT() {
  while (!mqttClient.connected()) {
    if (mqttClient.connect("esp8266-client", token, NULL)) {
      mqttStatus = "Conectado";
    } else {
      delay(5000);
    }
  }
}

// Envio periodico (cada 5 s) de telemetria:
String payload = "{\"temperature\":" + String(temp, 1) +
                 ",\"humidity\":" + String(hum, 1) + "}";
mqttClient.publish("v1/devices/me/telemetry", payload.c_str());
```

## Sensor de gas + clima (MQ-2 + DHT11 → ThingsBoard por HTTP)

ESP8266 con sensor de gas **MQ-2** y **DHT11**. Lee temperatura, humedad y nivel de gas, muestra un dashboard web tipo "monitor industrial" y envía la telemetría a ThingsBoard vía **HTTP REST** (`/api/v1/<token>/telemetry`). Sketch de 269 líneas, adjunto.

## Control de LED por web + RPC ThingsBoard

ESP8266 que controla el LED integrado tanto desde un panel web propio como desde **RPC de ThingsBoard** (método `setLED`, `checkStatus`). Publica el estado como telemetría y atributo para alimentar un widget *LED Indicator*. Sketch de 263 líneas, adjunto.

## Sketch de aproximación (HC-SR04 + relé)

Existe además un sketch de sensor **ultrasónico HC-SR04** que controla un relé y LEDs según un umbral de distancia, con panel web y envío a ThingsBoard por HTTP. Comparte la misma estructura que los anteriores.

## Firmware precompilado: ESP8266 Deauther

`esp8266_deauther_2.6.1_NODEMCU.bin` es **firmware precompilado (descarga)** del proyecto *ESP8266 Deauther* (v2.6.1, para placas NODEMCU). Se flashea directamente con `esptool.py` o NodeMCU Flasher; no es código fuente y no se embebe aquí.

```bash
# Ejemplo de flasheo (ajusta el puerto)
esptool.py --port /dev/ttyUSB0 write_flash 0x00000 esp8266_deauther_2.6.1_NODEMCU.bin
```

> [!warning] El deauther emite tramas de *deauthentication* contra redes WiFi. Su uso contra redes que no son tuyas es ilegal en la mayoría de jurisdicciones. Solo para auditoría autorizada y laboratorio.

---
📦 Archivos: ![[esp8266-monitor-paquetes-deauth.ino]] · ![[esp8266-rfid-rc522-pentest.ino]] · ![[esp8266-mq2-dht11-thingsboard.ino]] · ![[esp8266-control-led-web-thingsboard.ino]]
📦 Firmware: ![[esp8266_deauther_2.6.1_NODEMCU.bin]]

🔗 Relacionado: [[ThingsBoard en Docker]] · [[MQTT con Mosquitto en Debian]]
