---
tags:
  - linux
  - logs
  - auditoria
  - seguridad
aliases:
  - Logs y auditoría en Linux
  - journalctl y auditd
---

# Logs y auditoría

**Los logs son la memoria del sistema. Saber leerlos, filtrarlos y analizarlos es lo que separa a un sysadmin que adivina de uno que diagnostica.**

## Archivos de log principales

| Archivo | Contenido |
|---|---|
| `/var/log/syslog` | Log general del sistema (Debian/Ubuntu) |
| `/var/log/messages` | Log general (RHEL/CentOS) |
| `/var/log/auth.log` | Autenticaciones, sudo, SSH (Debian/Ubuntu) |
| `/var/log/secure` | Idem para RHEL/CentOS |
| `/var/log/kern.log` | Mensajes del kernel |
| `/var/log/dmesg` | Boot del kernel |
| `/var/log/dpkg.log` | Instalaciones de paquetes (Debian) |
| `/var/log/apt/` | Historial de apt |
| `/var/log/nginx/` | Access y error logs de nginx |
| `/var/log/mysql/` | Logs de MySQL/MariaDB |
| `/var/log/fail2ban.log` | Bans por fail2ban |

## Leer y seguir logs

```bash
tail -f /var/log/syslog             # Seguir en tiempo real
tail -n 100 /var/log/auth.log       # Últimas 100 líneas
head -n 50 /var/log/syslog          # Primeras 50 líneas
cat /var/log/syslog | less          # Paginar
less +F /var/log/syslog             # less siguiendo el archivo (q para parar, F para seguir)
```

## Filtrar con grep

```bash
grep "error" /var/log/syslog                    # Buscar "error"
grep -i "error" /var/log/syslog                 # Ignorar mayúsculas
grep -n "failed" /var/log/auth.log              # Con número de línea
grep -v "INFO" /var/log/syslog                  # Excluir líneas con "INFO"
grep -E "error|warning|critical" /var/log/syslog  # Múltiples patrones
grep -A 5 "FAILED" /var/log/auth.log            # 5 líneas después del match
grep -B 5 "FAILED" /var/log/auth.log            # 5 líneas antes del match
grep -C 5 "FAILED" /var/log/auth.log            # 5 líneas antes y después
grep -r "error" /var/log/nginx/                 # Recursivo en directorio
```

## Procesamiento con awk y sed

```bash
# awk: filtrar columnas y procesar texto
awk '{print $1, $4}' /var/log/nginx/access.log          # Columnas 1 y 4
awk '$9 == "404"' /var/log/nginx/access.log             # Filtrar por código HTTP
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20  # Top IPs

# sed: reemplazar y transformar
sed -n '/Jan 15/p' /var/log/syslog          # Solo líneas con "Jan 15"
sed 's/error/ERROR/g' archivo.log           # Reemplazar texto
sed '/^#/d' archivo.conf                     # Eliminar comentarios
```

## journalctl (systemd journal)

```bash
journalctl                              # Todo el journal
journalctl -f                           # En tiempo real
journalctl -u servicio                  # Logs de un servicio
journalctl -u servicio -f               # Seguir logs de un servicio
journalctl -b                           # Boot actual
journalctl -b -1                        # Boot anterior
journalctl --since "2024-01-01 08:00"
journalctl --since "1 hour ago"
journalctl --since "1 hour ago" --until "30 minutes ago"
journalctl -p err                       # Solo errores (emerg, alert, crit, err)
journalctl -p warning                   # Warnings y superiores
journalctl -k                           # Solo kernel
journalctl --no-pager | grep -i error   # Pipe a grep
journalctl -o json-pretty -u nginx      # Formato JSON
```

## Rotación de logs con logrotate

```bash
cat /etc/logrotate.conf             # Configuración global
ls /etc/logrotate.d/                # Configuraciones por aplicación
logrotate -d /etc/logrotate.conf    # Dry run (simular sin ejecutar)
logrotate -f /etc/logrotate.conf    # Forzar rotación ahora
```

### Ejemplo de configuración logrotate

logrotate no admite comentarios al final de la línea: van en su propia línea. La retención define cuánto tiempo se conserva la evidencia (NIST AU-11).

```bash
/var/log/miapp/*.log {
    # Rotar a diario y mantener 7 archivos
    daily
    rotate 7
    # Comprimir los viejos, desde el segundo
    compress
    delaycompress
    # Sin error si no existe; no rotar si está vacío
    missingok
    notifempty
    # Permisos del archivo nuevo
    create 0640 www-data adm
    postrotate
        systemctl reload nginx
    endscript
}
```

## Auditoría con auditd

```bash
# Instalar
apt install auditd

# Estado
systemctl status auditd
auditctl -s                         # Estado del subsistema de auditoría
auditctl -l                         # Reglas activas

# Agregar reglas
auditctl -w /etc/passwd -p wa -k cambios_passwd   # Monitorear escrituras en /etc/passwd
auditctl -w /etc/sudoers -p wa -k sudoers         # Monitorear sudoers
auditctl -a always,exit -F arch=b64 -S execve -k ejecuciones  # Registrar ejecuciones

# Consultar logs de auditoría
ausearch -k cambios_passwd          # Buscar por clave
ausearch -f /etc/passwd             # Buscar por archivo
ausearch -ua root                   # Acciones del usuario root
ausearch -ts today                  # Solo hoy
aureport --summary                  # Resumen
aureport --logins                   # Resumen de logins
aureport --failed                   # Solo fallos
```

## Monitoreo de accesos SSH

```bash
# Ver intentos fallidos de login
grep "Failed password" /var/log/auth.log
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Ver logins exitosos
grep "Accepted" /var/log/auth.log

# Ver intentos de login como root
grep "Invalid user\|Failed password for root" /var/log/auth.log

# Con journalctl
journalctl -u ssh --since "24 hours ago" | grep -i "failed\|invalid"
```

### Eventos de autenticación en auth.log (Ubuntu)

Cada intento de acceso o uso de sudo deja una línea con fecha, usuario, IP y resultado. Con `PasswordAuthentication no` dejan de aparecer las líneas `Failed password`; los fallos se ven con los mensajes de esta tabla.

| Evento | Línea típica |
|---|---|
| Login con llave exitoso | `sshd[...]: Accepted publickey for usr_admin from 203.0.113.10 port 51234 ssh2: ED25519 SHA256:...` |
| Llave correcta, falta el MFA | `sshd[...]: Partial publickey for usr_admin from 203.0.113.10 ...` |
| MFA exitoso | `sshd[...]: Accepted keyboard-interactive/pam for usr_admin from 203.0.113.10 ...` |
| Código MFA incorrecto | `sshd(pam_google_authenticator)[...]: Invalid verification code for usr_admin` |
| Cliente sin llave válida | `sshd[...]: Connection closed by authenticating user usr_admin 203.0.113.10 port 51234 [preauth]` |
| Usuario inexistente | `sshd[...]: Invalid user admin from 198.51.100.7 port 40022` |
| Intento como root | `sshd[...]: ROOT LOGIN REFUSED FROM 198.51.100.7 port 40022` |
| Grupo no autorizado | `sshd[...]: User invitado from 203.0.113.10 not allowed because none of user's groups are listed in AllowGroups` |
| sudo permitido | `sudo: usr_admin : TTY=pts/0 ; PWD=/home/usr_admin ; USER=root ; COMMAND=/usr/bin/apt update` |
| sudo con comando no autorizado | `sudo: usr_auditor : command not allowed ; ... ; COMMAND=/usr/bin/cat /etc/shadow` |
| sudo sin permisos | `sudo: invitado : user NOT in sudoers ; ... ; COMMAND=/usr/bin/ls /root` |

```bash
# Accesos SSH: exitosos, parciales, fallidos y rechazados
sudo grep -E "Accepted|Partial|Failed|Invalid|REFUSED|not allowed|Connection closed by authenticating" /var/log/auth.log
# Uso de sudo, permitido y denegado
sudo grep "sudo:" /var/log/auth.log
sudo grep -E "command not allowed|NOT in sudoers" /var/log/auth.log
# Códigos MFA incorrectos
sudo grep "Invalid verification code" /var/log/auth.log
# Sesiones: quién entró, quién falló, quién está conectado
last -a | head -20
sudo lastb -a | head -20
who
```

> [!tip] Cómo citar un evento en un informe
> Fecha y hora → servicio (`sshd`, `sudo`) → usuario → IP de origen → resultado. Es el contenido mínimo que exige NIST AU-3 (ver [[Marcos normativos (NIST SP 800-53, Well-Architected y CSA CCM)]]).

## Centralización de logs con rsyslog

```bash
# Ver configuración
cat /etc/rsyslog.conf
ls /etc/rsyslog.d/

# Enviar logs a servidor remoto (en rsyslog.conf del cliente)
*.* @192.168.1.100:514      # UDP
*.* @@192.168.1.100:514     # TCP

# Recibir logs en el servidor: descomentar en rsyslog.conf
# module(load="imudp")
# input(type="imudp" port="514")

systemctl restart rsyslog
```

## Casos de uso reales

**Investigar un intento de acceso no autorizado:**

```bash
grep "Invalid user" /var/log/auth.log | awk '{print $10,$13}' | sort | uniq -c | sort -rn
```

**Ver los últimos errores de cualquier servicio:**

```bash
journalctl -p err -b --no-pager | tail -30
```

**Cuánto espacio ocupan los logs:**

```bash
du -sh /var/log/* | sort -rh | head -10
journalctl --disk-usage
```

## Troubleshooting común

| Problema | Comando |
|---|---|
| Logs llenos, disco al 100% | `journalctl --vacuum-size=1G` |
| No encuentro cuándo falló algo | `journalctl -b -1 -p err` |
| Quiero ver accesos SSH de hoy | `journalctl -u ssh --since today` |
| Log de nginx vacío | `systemctl status nginx` (puede fallar antes de loguear) |
| auditd llena el disco | Revisar reglas demasiado amplias con `auditctl -l` |

> [!tip] Análisis de seguridad
> El conteo de IPs con `awk ... | sort | uniq -c | sort -rn` sobre `auth.log` revela rápidamente fuentes de ataques de fuerza bruta.

---
🔗 Relacionado: [[06 - Servicios con systemd]] · [[08 - Seguridad del servidor]]
