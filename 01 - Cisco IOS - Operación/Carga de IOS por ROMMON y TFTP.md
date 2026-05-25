---
tags:
  - cisco
  - operación
  - recuperación
  - rommon
  - tftp
aliases:
  - tftpdnld
  - Recuperar IOS
---

# Carga de Cisco IOS mediante ROMMON y TFTP

Procedimiento para instalar o recuperar una imagen Cisco IOS (`.bin`) usando el modo **ROMMON** y un **servidor TFTP** en un computador.

> [!warning] Este procedimiento borra completamente la memoria flash del router.

## Concepto

ROMMON no puede leer archivos directamente desde el PC, por lo que el computador actúa como **servidor TFTP temporal**.

```
PC con TFTP  --->  Router Cisco
```

## Requisitos
Cable consola · cable Ethernet · imagen IOS `.bin` · software TFTP · conectividad PC↔router.

**TFTP recomendado (Windows):** Tftpd64, Tftpd32, SolarWinds TFTP Server.

## Pasos

1. **Servidor TFTP**: crear carpeta `C:\TFTP-Root\` y copiar la imagen (ej. `2600.bin`).
2. **IP del PC** (ejemplo): PC TFTP `192.168.20.100`, Router `192.168.20.1`.
3. **Conectar** router y PC (cable directo o switch).
4. **Entrar a ROMMON**: reiniciar e interrumpir el arranque (ver tabla Break).
5. **Configurar parámetros de red** en ROMMON:

```
rommon 1 > IP_ADDRESS=192.168.20.1
rommon 2 > IP_SUBNET_MASK=255.255.255.0
rommon 3 > DEFAULT_GATEWAY=192.168.20.100
rommon 4 > TFTP_SERVER=192.168.20.100
rommon 5 > TFTP_FILE=c2900-universalk9-mz.SPA.155-3.M4a.bin
```

6. **Descargar la IOS**: `rommon 6 > tftpdnld`
7. **Confirmar** el borrado de flash con `y`.
8. **Esperar** la transferencia (⚠️ no apagar el equipo).
9. **Reiniciar**: `rommon > reset` (o `boot`).

## Combinaciones Break (interrumpir arranque)

| Software | Combinación |
|---|---|
| PuTTY / Tera Term / HyperTerminal | `Ctrl + Break` |
| Laptop sin Break | `Fn + Pause/Break` |
| SecureCRT | `Ctrl + Shift + 6` luego `b` |
| Minicom | `Ctrl + A` luego `F` |
| macOS `screen` | `Ctrl + A` luego `Ctrl + \` |

## Servidor TFTP en Linux

```bash
sudo apt install tftpd-hpa
sudo mkdir -p /var/lib/tftpboot
sudo chmod 777 /var/lib/tftpboot
sudo systemctl start tftpd-hpa
sudo cp archivo.bin /var/lib/tftpboot/
```

> [!info] Comando `tftpdnld`
> Descarga una imagen IOS desde un servidor TFTP a la flash del router. Útil ante IOS corrupta, flash vacía o recuperación de desastres.

---
🔗 Relacionado: [[Recuperación de contraseña por ROMMON]] · [[Recuperación de equipos Cisco (referencia rápida)]]
