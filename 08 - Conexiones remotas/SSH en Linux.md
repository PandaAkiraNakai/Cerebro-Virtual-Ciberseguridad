---
tags:
  - linux
  - ssh
  - acceso-remoto
  - seguridad
aliases:
  - OpenSSH Linux
  - SSH Linux
---

# SSH en Linux

Instalación y endurecimiento de **OpenSSH** en Linux.

## Instalación

```bash
# Debian / Ubuntu y derivados
sudo apt update && sudo apt install -y openssh-server

# RHEL / CentOS / Fedora
sudo dnf install -y openssh-server

# Arch Linux
sudo pacman -S openssh
```

## Servicio

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
```

## Configuración — `/etc/ssh/sshd_config`

```ini
Port 2222                      # cambiar el puerto por defecto
PermitRootLogin no             # deshabilitar acceso root directo
AllowUsers usuario1 usuario2   # solo usuarios específicos
PasswordAuthentication no      # forzar autenticación por clave
Protocol 2                     # solo SSH v2
```

Aplicar cambios:

```bash
sudo systemctl restart ssh
```

## Claves SSH

```bash
ssh-keygen -t rsa -b 4096          # generar par de claves
ssh-copy-id usuario@servidor       # copiar la clave pública
ssh -p 2222 usuario@servidor       # conectar
```

## Seguridad adicional

```bash
# Fail2Ban contra fuerza bruta
sudo apt install fail2ban    # Debian/Ubuntu
sudo dnf install fail2ban    # RHEL/CentOS
```

- Usar autenticación por claves en vez de contraseña.
- Deshabilitar protocolos inseguros.

## Desinstalación

```bash
sudo apt remove --purge -y openssh-server   # Debian/Ubuntu
sudo dnf remove -y openssh-server           # RHEL/CentOS
sudo pacman -Rns openssh                     # Arch
```

## Referencias
- [Documentación oficial de OpenSSH](https://www.openssh.com/manual.html)

---
🔗 Relacionado: [[SSH en Raspberry Pi OS]] · [[SSH en Cisco]]
