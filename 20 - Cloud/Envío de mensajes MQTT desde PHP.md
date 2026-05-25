---
tags:
  - cloud
  - mqtt
  - php
  - iot
aliases:
  - MQTT Sender PHP
  - Publicar MQTT con PHP
---

# Envío de mensajes MQTT desde PHP

**Pequeña interfaz web en PHP que publica mensajes en un broker MQTT (Mosquitto) mediante `mosquitto_pub`, y registra cada envío en un archivo de log. Útil para probar tópicos MQTT de proyectos IoT desde el navegador, sin programar un cliente dedicado.**

## Cómo funciona

1. Un formulario HTML pide **IP del broker**, **tópico** y **mensaje**.
2. Al enviar, PHP ejecuta `mosquitto_pub` por la shell del servidor.
3. Cada envío se anota con marca de tiempo en un archivo de log que la página muestra.

> [!warning] Inyección de comandos
> Este script pasa la entrada del usuario directamente a `shell_exec` sin sanitizarla, lo que permite **inyección de comandos**. Es válido solo como demostración en un entorno controlado. En producción valida/escapa la entrada (`escapeshellarg`) o usa una librería cliente MQTT nativa de PHP.

> [!info] Requisito en el servidor
> El binario `mosquitto-clients` (que provee `mosquitto_pub`) debe estar instalado y accesible para el usuario del servidor web.

## Versión 1 (ruta de log relativa al script)

```php
<?php
// Configuración
$logFile = __DIR__ . '/mqtt_log.txt';  // ruta absoluta basada en el directorio del script
$maxLogEntries = 20;

// Procesar el envío del mensaje
if (isset($_POST['send'])) {
    $ip = $_POST['ip'];
    $topic = $_POST['topic'];
    $message = $_POST['message'];
    $timestamp = date('Y-m-d H:i:s');

    // Comando para publicar el mensaje MQTT
    $cmd = "mosquitto_pub -h $ip -t '$topic' -m '$message' 2>&1";
    $output = shell_exec($cmd);

    // Registrar el envío
    $logEntry = "[$timestamp] Enviado a $ip - Tópico: $topic - Mensaje: $message";
    if (!empty($output)) {
        $logEntry .= " (Error: " . trim($output) . ")";
    }

    file_put_contents($logFile, $logEntry . PHP_EOL, FILE_APPEND);

    // Redirigir para evitar reenvío al recargar
    header("Location: " . $_SERVER['PHP_SELF']);
    exit();
}

// Mostrar el log (últimas N entradas, más recientes primero)
if (file_exists($logFile)) {
    $logContent = file($logFile, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    if ($logContent === false) {
        echo "Error al leer el archivo de log.";
    } else {
        $logContent = array_reverse($logContent);
        $logContent = array_slice($logContent, 0, $maxLogEntries);
        foreach ($logContent as $entry) {
            echo htmlspecialchars($entry) . "<br>";
        }
    }
} else {
    if (@file_put_contents($logFile, '') === false) {
        echo "No se pudo crear el archivo de log. Verifica permisos en: " . htmlspecialchars(__DIR__);
    } else {
        echo "Archivo de log creado. Envía tu primer mensaje.";
    }
}
?>
```

## Versión 2 (ruta de log fija + estilos)

La segunda variante es funcionalmente igual pero usa una ruta absoluta fija para el log y agrega estilo a la interfaz. La lógica PHP relevante:

```php
<?php
$logFile = '/var/www/html/mqtt_log.txt';
$maxLogEntries = 20;

if (isset($_POST['send'])) {
    $ip = $_POST['ip'];
    $topic = $_POST['topic'];
    $message = $_POST['message'];
    $timestamp = date('Y-m-d H:i:s');

    $cmd = "mosquitto_pub -h $ip -t '$topic' -m '$message' 2>&1";
    $output = shell_exec($cmd);

    $logEntry = "[$timestamp] Enviado a $ip - Tópico: $topic - Mensaje: $message";
    if (!empty($output)) {
        $logEntry .= " (Error: " . trim($output) . ")";
    }

    file_put_contents($logFile, $logEntry . PHP_EOL, FILE_APPEND);
    header("Location: " . $_SERVER['PHP_SELF']);
    exit();
}

if (file_exists($logFile)) {
    $logContent = file($logFile, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    if ($logContent === false) {
        echo "Error al leer el archivo de log.";
    } else {
        $logContent = array_reverse($logContent);
        $logContent = array_slice($logContent, 0, $maxLogEntries);
        foreach ($logContent as $entry) {
            echo htmlspecialchars($entry) . "<br>";
        }
    }
} else {
    echo "Archivo de log no accesible.";
}
?>
```

> [!tip] `__DIR__` vs. ruta fija
> La versión 1 usa `__DIR__` para que el log quede junto al script sin importar dónde se despliegue; la versión 2 fija `/var/www/html/`. La primera es más portable; en ambos casos el directorio debe tener permisos de escritura para el usuario del servidor web.

## Formulario HTML

Ambas versiones comparten un formulario con tres campos (`ip`, `topic`, `message`) enviados por POST. Ejemplo de placeholders sugeridos:

- Broker IP: `<ip_del_broker>` (ej. `192.168.1.100`)
- Tópico: `casa/sala/luz`
- Mensaje: `encender`

---
📎 Guías: scripts originales `mqtt-mensaje-1.php` y `mqtt-mensaje-2.php` (embebidos arriba).

🔗 Relacionado: [[MySQL + phpMyAdmin]] · [[AWS - VPC, subredes, balanceo y autoescalado]]
