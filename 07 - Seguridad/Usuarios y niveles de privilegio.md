---
tags:
  - cisco
  - seguridad
  - autenticación
  - privilegios
aliases:
  - Niveles de privilegio
  - Privilege levels
---

# Usuarios y niveles de privilegio en routers Cisco

Configurar distintos niveles de acceso permite **segmentar privilegios** (administradores, usuarios avanzados, solo lectura) — buena práctica de seguridad para limitar lo que cada usuario puede hacer.

## Niveles de privilegio

Cisco IOS tiene **16 niveles (0–15)**:

| Nivel | Acceso por defecto |
|---|---|
| **0** | Comandos básicos (`logout`, `exit`) |
| **1** | Comandos de visualización (`show`) |
| **15** | Acceso completo a todos los comandos |

Ejemplo de segmentación:
- **Privilegio 15** → Administrador (acceso completo).
- **Privilegio 5** → Usuario avanzado (algunos comandos, ej. `ping`).
- **Privilegio 3** → Solo lectura (solo visualización).

## Crear usuarios

**Administrador (15):**

```cisco
Router(config)# username admin privilege 15 secret [contraseña]
Router(config)# enable secret [contraseña_enable]
```

**Usuario avanzado (5):**

```cisco
Router(config)# privilege exec level 5 show running-config
Router(config)# privilege exec level 5 ping
Router(config)# username usuario_avanzado privilege 5 secret [contraseña]
```

**Solo lectura (3):**

```cisco
Router(config)# privilege exec level 3 show running-config
Router(config)# username solo_lectura privilege 3 secret [contraseña]
```

## Acceso remoto por SSH

```cisco
Router(config)# line vty 0 4
Router(config-line)# login local
Router(config-line)# transport input ssh
Router(config)# ip domain-name [dominio]
Router(config)# crypto key generate rsa
Router(config)# ip ssh version 2
```

## Verificación

```cisco
Router# show running-config | include username   ! usuarios configurados
Router# show privilege                           ! nivel actual
```

Probar desde un equipo externo:

```bash
ssh [usuario]@[IP_del_router]
```

---
📎 Guía completa: ![[Usuarios-y-Privilegios-Cisco.pdf]]

🔗 Relacionado: [[AAA en Cisco]] · [[SSH en Cisco]] · [[Acceso y autenticación segura en Cisco]]
