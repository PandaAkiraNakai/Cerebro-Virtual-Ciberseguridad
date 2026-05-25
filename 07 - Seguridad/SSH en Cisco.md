---
tags:
  - cisco
  - seguridad
  - ssh
  - acceso-remoto
aliases:
  - SSH Cisco
  - SSH router Cisco
---

# SSH en dispositivos Cisco

**SSH (Secure Shell)** permite administración remota **cifrada**, reemplazando a Telnet (que viaja en claro). Usa cifrado (AES) y autenticación (contraseña o claves RSA), con verificación de integridad (MAC).

## ¿Cómo funciona?
1. **Autenticación**: por contraseña o por par de claves RSA (pública en el servidor, privada en el cliente).
2. **Cifrado**: todos los datos se cifran (ej. AES).
3. **Integridad**: códigos MAC detectan alteraciones en tránsito.

## Configuración paso a paso

```cisco
! 1. Contraseña secreta del modo privilegiado
enable secret strongpassword

! 2. Hostname y dominio (necesario para generar claves)
hostname R1
ip domain-name ejemplo.com

! 3. Usuario local con privilegio máximo
username admin privilege 15 secret yourpassword

! 4. Generar claves RSA (2048 bits recomendado)
crypto key generate rsa general-keys modulus 2048

! 5. Líneas VTY: solo SSH + autenticación local
line vty 0 4
 transport input ssh
 login local
 exit

! 6. Forzar SSH versión 2 (más segura)
ip ssh version 2

! 7. (Opcional) Cifrar las claves RSA
crypto key encrypt rsa
```

## Endurecimiento adicional

```cisco
ip ssh time-out 60                 ! timeout de autenticación
ip ssh authentication-retries 2    ! máximo de reintentos
```

## Verificación

```cisco
show ip ssh        ! configuración de SSH
show ssh           ! sesiones activas
```

## Probar desde una PC (ej. Packet Tracer)

```bash
ssh -l admin 192.168.1.1
```

> [!tip] Restringir el acceso
> Combina con una [[ACL estándar]] aplicada a las VTY (`access-class ... in`) para permitir SSH solo desde IPs de administración. Ver [[Acceso y autenticación segura en Cisco]].

---
📎 Guía completa: ![[SSH-Router-Cisco.pdf]]

🔗 Relacionado: [[Acceso y autenticación segura en Cisco]] · [[Usuarios y niveles de privilegio]] · [[AAA en Cisco]] · [[SSH en Linux]]
