---
tags:
  - cisco
  - operación
  - recuperación
  - tftp
aliases:
  - Cheatsheet recuperación Cisco
---

# Recuperación de equipos Cisco — referencia rápida

Referencia compacta para recuperación por TFTP. La guía detallada está en [[Carga de IOS por ROMMON y TFTP]].

## Routers Cisco (ej. 1841) vía TFTP

```
IP_ADDRESS=192.168.0.2
IP_SUBNET_MASK=255.255.255.0
DEFAULT_GATEWAY=192.168.0.1
TFTP_SERVER=192.168.0.100
TFTP_FILE=c1841-adventerprisek9-mz.124-25f.bin
tftpdnld
boot flash:archivo.bin
```

## Servidor TFTP en Ubuntu

```bash
sudo apt install tftpd-hpa
sudo mkdir -p /var/lib/tftpboot
sudo chmod 777 /var/lib/tftpboot
sudo systemctl start tftpd-hpa
sudo cp archivo.bin /var/lib/tftpboot/
```

| Paso | Comando | Descripción |
|---|---|---|
| 1 | `IP_ADDRESS=192.168.0.2` | IP del router |
| 2 | `IP_SUBNET_MASK=255.255.255.0` | Máscara |
| 3 | `DEFAULT_GATEWAY=192.168.0.1` | Gateway |
| 4 | `TFTP_SERVER=192.168.0.100` | IP del servidor TFTP |
| 5 | `TFTP_FILE=...bin` | Nombre exacto del archivo |
| 6 | `tftpdnld` | Iniciar transferencia |
| 7 | `boot flash:archivo.bin` | Bootear tras transferir |

---
🔗 Relacionado: [[Carga de IOS por ROMMON y TFTP]] · [[Recuperación de contraseña por ROMMON]]
