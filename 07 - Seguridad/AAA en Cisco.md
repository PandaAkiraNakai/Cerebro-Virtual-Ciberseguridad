---
tags:
  - cisco
  - seguridad
  - aaa
  - autenticación
aliases:
  - AAA
  - Authentication Authorization Accounting
---

# AAA en Cisco

**AAA** controla el acceso a dispositivos de red mediante tres funciones:

- **Autenticación** → ¿quién eres? (verifica identidad)
- **Autorización** → ¿qué puedes hacer? (define permisos)
- **Contabilización (Accounting)** → ¿qué hiciste? (registra actividad)

## Habilitar AAA (obligatorio)

```cisco
aaa new-model
```

## Usuarios con niveles de privilegio

```cisco
username USER     privilege 1  secret cisco1     ! básico
username SUPPORT  privilege 5  secret cisco5     ! soporte
username JR-ADMIN privilege 10 secret cisco10    ! admin junior
username ADMIN    privilege 15 secret cisco123   ! admin total
```

> Ver detalle de niveles en [[Usuarios y niveles de privilegio]].

## Personalizar comandos por nivel

```cisco
privilege exec level 5 ping     ! solo nivel 5+ puede usar ping
privilege exec level 10 reload  ! solo nivel 10+ puede recargar
```

## Contraseñas de cambio entre niveles

```cisco
enable secret level 5 cisco5
enable secret level 10 cisco10
enable secret cisco15            ! nivel 15 (máximo)
```

## Políticas AAA

```cisco
aaa authentication login default local                 ! autentica con base local
aaa authorization exec default local                   ! autoriza nivel según usuario
aaa accounting exec default start-stop group local     ! registra inicio/fin de sesión
```

## Aplicar AAA a las líneas

```cisco
line con 0
 logging synchronous
 login authentication default

line vty 0 15
 transport input telnet           ! solo laboratorio; en producción usar SSH
 login authentication default
 authorization exec default
```

## Verificación

```cisco
show running-config | section aaa
show privilege
show aaa sessions
show users
```

> [!warning] Telnet vs SSH
> `transport input telnet` envía credenciales en claro. En producción usa **SSH**: ver [[SSH en Cisco]].

---
🔗 Relacionado: [[Usuarios y niveles de privilegio]] · [[Acceso y autenticación segura en Cisco]] · [[SSH en Cisco]]
