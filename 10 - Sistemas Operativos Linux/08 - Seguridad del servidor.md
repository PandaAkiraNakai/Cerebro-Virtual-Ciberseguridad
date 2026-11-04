---
tags:
  - linux
  - seguridad
  - hardening
aliases:
  - Seguridad del servidor Linux
  - Hardening de servidores
---

# Seguridad del servidor

**Un servidor expuesto sin hardening es una invitación abierta. Estas son las prácticas y herramientas fundamentales para reducir la superficie de ataque.**

## SSH — Configuración segura

### Llaves SSH paso a paso

Se usa criptografía asimétrica: dos llaves que funcionan juntas, como una llave y su candado.

| Archivo | Tipo | Dónde queda | ¿Se envía? |
|---|---|---|---|
| `~/.ssh/ana` | Privada (la llave) | Solo en tu PC | **Nunca** |
| `~/.ssh/ana.pub` | Pública (el candado) | PC y servidor (`~/.ssh/authorized_keys`) | Sí, sin riesgo |

```text
PC                                        Servidor
1. ssh-keygen crea ana y ana.pub
2. Envía SOLO ana.pub            ───►     La guarda en ~/.ssh/authorized_keys
3. ssh -i ana                    ◄──►     Desafío: el PC lo firma con la privada
4.                                        Verifica la firma con la pública → entra
```

La privada nunca viaja por la red: el servidor solo comprueba que la tienes.

```bash
# 1. En tu PC, no en el servidor (-f da un nombre por usuario o servidor)
ssh-keygen -t ed25519 -f ~/.ssh/ana          # la frase de paso protege la llave si te roban el archivo

# 2. Enviar solo la pública (pide una vez la contraseña del usuario)
ssh-copy-id -i ~/.ssh/ana.pub ana@IP

# 3. Conectarse: debe pedir la frase de la llave, no la contraseña del usuario
ssh -i ~/.ssh/ana ana@IP
```

En Windows (PowerShell), donde no existe `ssh-copy-id`:

```powershell
type $env:USERPROFILE\.ssh\ana.pub | ssh ana@IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

En AWS EC2 el login con contraseña viene apagado y ninguna de las dos opciones funciona: la pública se agrega desde una sesión que ya tenga acceso.

### Hardening con un archivo propio

Crea `/etc/ssh/sshd_config.d/10-seguridad.conf` en vez de tocar `sshd_config`. En Ubuntu cloud existe `50-cloud-init.conf`, que puede reactivar `PasswordAuthentication`; en OpenSSH **gana el primer valor leído**, y el `10-` se lee antes.

```bash
sudo nano /etc/ssh/sshd_config.d/10-seguridad.conf
```

```text
# Nunca login directo como root
PermitRootLogin no
# Solo llaves, sin contraseñas
PasswordAuthentication no
PubkeyAuthentication yes
# 3 intentos por conexión y 20 s para autenticarse
MaxAuthTries 3
LoginGraceTime 20
# Sin reenvío gráfico si no se usa
X11Forwarding no
# Mensaje legal antes del login
Banner /etc/ssh/banner.txt
# Detecta conexiones caídas (no cierra sesiones inactivas: para eso, TMOUT)
ClientAliveInterval 300
ClientAliveCountMax 2
# Cambiar el puerto es opcional; en la nube ábrelo antes en el firewall o el Security Group
# Port 2222
```

> [!warning] Sin comentarios al final de la línea
> `PermitRootLogin no  # comentario` hace fallar el servicio. Los comentarios van en su propia línea.

```bash
sudo sshd -t                     # valida la sintaxis: sin salida = correcto
sudo systemctl restart ssh       # en Ubuntu y Debian el servicio se llama ssh
sudo sshd -T | grep -Ei "permitroot|passwordauth|maxauth|authenticationmethods|allowgroups"   # configuración efectiva
```

> [!warning] Verifica antes de cerrar la puerta
> Mantén una sesión abierta y prueba desde una **segunda** terminal. Confirma que entras con llave antes de poner `PasswordAuthentication no`; si algo falla, lo corriges desde la sesión abierta.

### MFA con TOTP

Cualquier app compatible sirve: Google Authenticator, Microsoft Authenticator, Aegis.

```bash
sudo apt install libpam-google-authenticator
google-authenticator        # como el usuario, sin sudo
```

Respuestas recomendadas: `y` (basado en tiempo), escanear el QR y guardar los códigos de emergencia, `y` (actualizar archivo), `y` (no reutilizar códigos), `n` (no ampliar la ventana), `y` (limitar intentos).

En `/etc/pam.d/sshd`:

```text
# Comenta esta línea cerca del inicio; si no, además del código pide la contraseña
#@include common-auth
# Al final del archivo
auth required pam_google_authenticator.so
```

Sin `nullok` el MFA es obligatorio y un usuario no enrolado no entra. Durante la transición usa `auth required pam_google_authenticator.so nullok` y quítalo cuando todos estén enrolados.

En `10-seguridad.conf`:

```text
UsePAM yes
KbdInteractiveAuthentication yes
# Exige llave Y código; sin esta línea la llave sola basta y el MFA nunca se pide
AuthenticationMethods publickey,keyboard-interactive
```

```bash
sudo sshd -t && sudo systemctl restart ssh
ssh -i ~/.ssh/ana ana@IP                      # pide la frase de la llave y luego "Verification code"
ssh -o PubkeyAuthentication=no ana@IP         # evidencia: debe dar Permission denied
sudo journalctl -u ssh --since today --no-pager
```

El TOTP depende de la hora: si los códigos fallan siempre, revisa `timedatectl` en el servidor y la hora del teléfono.

### Acceso por grupo, IP y sesiones

```text
# Solo estos grupos entran por SSH (incluye el del administrador o quedarás fuera)
AllowGroups sysops auditores
# Por usuario e IP: USUARIO@IP o USUARIO@RED/MÁSCARA; los no listados quedan bloqueados
AllowUsers usr_admin usr_auditor@203.0.113.10

# MFA solo para un grupo. Los bloques Match van AL FINAL del archivo
Match Group sysops
    AuthenticationMethods publickey,keyboard-interactive
```

`AllowGroups` y `AllowUsers` se aplican juntos: el usuario debe cumplir ambos.

```bash
# Máximo 2 sesiones simultáneas para el grupo auditores (pam_limits)
echo "@auditores hard maxlogins 2" | sudo tee -a /etc/security/limits.conf
# Cerrar la shell tras 10 minutos sin actividad
echo 'readonly TMOUT=600; export TMOUT' | sudo tee /etc/profile.d/99-tmout.sh

who; w                        # quién está conectado y qué hace
last -a | head -20            # historial de sesiones con IP
sudo lastb -a | head -20      # intentos fallidos
lastlog                       # último acceso de cada usuario
loginctl list-sessions
sudo pkill -KILL -t pts/1     # cerrar a la fuerza la sesión de pts/1
```

## Fail2ban — Bloqueo automático de ataques

```bash
# Instalar
apt install fail2ban

# Estado
systemctl status fail2ban
fail2ban-client status              # Ver jails activas
fail2ban-client status sshd         # Estado de la jail SSH

# Gestión de IPs baneadas
fail2ban-client set sshd unbanip 1.2.3.4    # Desbanear IP
fail2ban-client banned              # Ver todas las IPs baneadas
```

### Configuración en `/etc/fail2ban/jail.local`

```ini
[DEFAULT]
# Duración del ban, ventana para contar intentos e intentos antes del ban
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
# ssh = puerto 22; si lo cambiaste, pon el número (ej. 2222)
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h
```

```bash
systemctl restart fail2ban
```

## UFW — Firewall simplificado

```bash
# Estado
ufw status
ufw status verbose
ufw status numbered         # Con números de reglas

# Configuración básica (servidor web)
ufw default deny incoming   # Bloquear todo lo entrante por defecto
ufw default allow outgoing  # Permitir todo lo saliente

ufw allow ssh               # Puerto 22
ufw allow 2222/tcp          # SSH en puerto personalizado
ufw allow 80/tcp            # HTTP
ufw allow 443/tcp           # HTTPS
ufw allow from 10.0.0.0/24  # Permitir red interna completa
ufw allow from 10.0.0.5 to any port 5432   # PostgreSQL solo desde IP específica

# Activar
ufw enable

# Eliminar regla
ufw delete allow 80/tcp
ufw delete 3                # Por número (de ufw status numbered)

# Logs
ufw logging on
tail -f /var/log/ufw.log
```

## Actualizaciones de seguridad

```bash
# Debian/Ubuntu
apt update
apt upgrade
apt list --upgradable
apt-get dist-upgrade         # Incluye actualizaciones de kernel

# Actualizaciones automáticas de seguridad
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades
cat /etc/apt/apt.conf.d/50unattended-upgrades

# RHEL/CentOS
yum update
dnf update
dnf check-update
```

## Auditoría de seguridad con Lynis

```bash
# Instalar
apt install lynis

# Auditar el sistema completo
lynis audit system

# El reporte queda en /var/log/lynis.log
# El reporte detallado en /var/log/lynis-report.dat
```

## Verificar rootkits

```bash
# chkrootkit
apt install chkrootkit
chkrootkit

# rkhunter
apt install rkhunter
rkhunter --update
rkhunter --check
rkhunter --check --skip-keypress      # Sin pausas interactivas
```

## Gestión de certificados SSL/TLS

```bash
# Ver certificado de un sitio
openssl s_client -connect dominio.com:443
echo | openssl s_client -connect dominio.com:443 2>/dev/null | openssl x509 -noout -dates

# Verificar certificado local
openssl x509 -in certificado.crt -text -noout
openssl x509 -in certificado.crt -noout -enddate  # Fecha de expiración

# Generar certificado autofirmado (para uso interno)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Let's Encrypt con certbot
apt install certbot
certbot certonly --standalone -d dominio.com
certbot renew                           # Renovar todos los certificados
certbot renew --dry-run                 # Probar renovación sin ejecutar
```

## Monitoreo de archivos críticos

```bash
# Ver archivos SUID/SGID (posibles vectores de escalación)
find / -perm /4000 2>/dev/null          # SUID
find / -perm /2000 2>/dev/null          # SGID

# Archivos world-writable (peligrosos)
find / -perm -o+w -not -path "/proc/*" 2>/dev/null

# Archivos sin dueño
find / -nouser -o -nogroup 2>/dev/null

# Verificar integridad de paquetes instalados
debsums -c                              # Verifica checksums (Debian)
rpm -Va                                 # Idem en RHEL/CentOS
```

## Hardening adicional

```bash
# Deshabilitar servicios innecesarios
systemctl list-units --type=service --state=running
systemctl disable --now servicio_innecesario

# Limitar acceso a comandos sensibles
chmod 700 /usr/bin/top                  # Solo root (ejemplo)

# Configurar umask seguro
echo "umask 027" >> /etc/profile        # Nuevos archivos: 640, directorios: 750

# Bloquear acceso a /proc/PID de otros usuarios (en /etc/fstab)
proc /proc proc defaults,hidepid=2 0 0

# Limitar core dumps
echo "* hard core 0" >> /etc/security/limits.conf
```

## Casos de uso reales

**Hardening rápido de un servidor nuevo:**

```bash
apt update && apt upgrade -y
ufw default deny incoming && ufw allow 22/tcp && ufw enable
apt install fail2ban -y && systemctl enable --now fail2ban
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sshd -t && systemctl restart ssh
```

**Verificar si alguien entró al servidor:**

```bash
last -n 20
grep "Accepted" /var/log/auth.log | tail -20
journalctl -u ssh --since "24 hours ago"
```

## Troubleshooting común

| Problema | Comando |
|---|---|
| Me baneé a mí mismo con fail2ban | Acceso físico/consola → `fail2ban-client set sshd unbanip TU_IP` |
| SSH no acepta la clave | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| Puerto SSH bloqueado por UFW | `ufw status` desde consola local |
| Certificado SSL expirado | `certbot renew` o `openssl x509 -noout -enddate -in cert.pem` |
| SSH falla al reiniciar | `sshd -t` muestra la línea con error |
| Ya no pide el código MFA | Falta `AuthenticationMethods publickey,keyboard-interactive` |
| Pide contraseña además del código | Comenta `@include common-auth` en `/etc/pam.d/sshd` |
| Quedé fuera de una instancia EC2 | EC2 Instance Connect o consola serial desde AWS |

---
🔗 Relacionado: [[07 - Logs y auditoría]] · [[05 - Usuarios, grupos y permisos]] · [[09 - Backups y recuperación]] · [[SSH en Linux]] · [[Marcos normativos (NIST SP 800-53, Well-Architected y CSA CCM)]]
