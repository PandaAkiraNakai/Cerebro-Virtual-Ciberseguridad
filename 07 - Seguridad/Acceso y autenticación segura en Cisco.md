---
tags:
  - cisco
  - seguridad
  - hardening
  - autenticación
aliases:
  - Hardening Cisco
  - Acceso seguro Cisco
---

# Configuración segura de acceso y autenticación en Cisco

Endurecimiento del acceso a dispositivos Cisco: contraseñas cifradas, control de intentos de login, SSH, ACLs y monitoreo.

## 1. Cifrado de contraseñas y políticas

```cisco
service password-encryption          ! cifra todas las contraseñas almacenadas
security passwords min-length 8       ! longitud mínima de 8 caracteres
```

## 2. Control de intentos de login

```cisco
login block-for 120 attempts 3 within 60   ! bloquea 120s tras 3 fallos en 60s
login delay 10                              ! 10s entre intentos
login on-success log                        ! registra logins exitosos
login on-failure log                        ! registra intentos fallidos
```

## 3. Contraseña enable (SCRYPT, tipo 9)

```cisco
enable algorithm-type scrypt secret cisco12345
```

## 4. Gestión de usuarios

```cisco
username Bob algorithm-type scrypt secret cisco54321
```

## 5. ACL para acceso restringido

```cisco
ip access-list standard PERMIT-ADMIN
 remark Permit only Administrative hosts
 permit 192.168.10.10
 permit 192.168.11.10
login quiet-mode access-class PERMIT-ADMIN
```

## 6. SSH

```cisco
ip domain-name span.com
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 2
```

> Ver guía dedicada: [[SSH en Cisco]] (recomienda módulo **2048** bits).

## 7. Líneas VTY

```cisco
line vty 0 4
 password cisco123
 login local
 exec-timeout 5 30          ! timeout de sesión (5 min 30 s)
 transport input ssh        ! solo SSH
```

## 8. Monitoreo y diagnóstico

```cisco
show login failures    ! intentos fallidos de login
show ip ssh            ! configuración SSH
```

---
🔗 Relacionado: [[SSH en Cisco]] · [[AAA en Cisco]] · [[Usuarios y niveles de privilegio]] · [[ACL estándar]] · [[Seguridad en la Capa 2]]
