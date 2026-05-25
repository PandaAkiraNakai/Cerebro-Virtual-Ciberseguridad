---
tags:
  - linux
  - ssh
  - raspberry-pi
  - acceso-remoto
aliases:
  - SSH Raspberry Pi
  - raspi-config SSH
---

# SSH en Raspberry Pi OS

## Instalación

Raspberry Pi OS suele incluir OpenSSH; si no:

```bash
sudo apt update && sudo apt install -y openssh-server
```

## Servicio

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
```

## Habilitar SSH desde `raspi-config`

```bash
sudo raspi-config
```

Ve a **Interfacing Options → SSH** y actívalo.

## Configuración — `/etc/ssh/sshd_config`

```ini
Port 2222
PermitRootLogin no
AllowUsers usuario1 usuario2
PasswordAuthentication no
```

Aplicar: `sudo systemctl restart ssh`

## Claves SSH

```bash
ssh-keygen -t ed25519
ssh-copy-id usuario@raspberrypi
ssh -p 2222 usuario@raspberrypi
```

## Seguridad adicional

```bash
sudo apt install fail2ban       # contra fuerza bruta
sudo ufw allow 2222/tcp         # firewall
sudo ufw enable
```

## Referencias
- [Documentación oficial de OpenSSH](https://www.openssh.com/manual.html)
- [Guía de Raspberry Pi](https://www.raspberrypi.com/documentation/)

---
🔗 Relacionado: [[SSH en Linux]] · [[SSH en Cisco]]
