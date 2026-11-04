---
tags:
  - cisco
  - telnet
  - acceso-remoto
aliases:
  - Telnet Cisco
  - Líneas VTY
---

# Telnet en Cisco IOS

**Telnet** abre una sesión de terminal remota sobre TCP/IP (puerto 23). Para que un router o switch la acepte hay que configurar las **líneas VTY** (*Virtual Terminal Lines*), los canales virtuales que IOS reserva para sesiones remotas. Sin ellas, el equipo rechaza la conexión.

> [!danger] Solo para laboratorio
> Telnet transmite todo en texto plano, contraseña incluida; una captura en [[Wireshark]] la revela. En producción usa [[SSH en Cisco|SSH]].

## Requisitos

- Acceso a configuración global.
- Al menos una interfaz con IP activa.
- Conectividad IP entre tu PC y el equipo.

## Configuración

```cisco
enable
configure terminal
line vty 0 4                 ! 5 sesiones simultáneas (0-15 donde exista)
 password cisco              ! contraseña de la línea
 login                       ! sin esto la contraseña no se pide
 transport input telnet      ! o "ssh telnet" para ambos
end
write memory
```

## Verificación

```bash
telnet 192.168.1.1
```

El equipo pide la contraseña y entrega el modo EXEC de usuario (`Router>`).

## `login` frente a `login local`

| Comando | Autenticación |
|---|---|
| `login` | Contraseña única definida con `password` en la línea |
| `login local` | Usuarios creados con `username … secret` en la configuración global |

`login local` es preferible: permite usuarios individuales con distintos privilegios (ver [[Usuarios y niveles de privilegio]]).

## Siguiente paso: migrar a SSH

```cisco
ip domain-name lab.local
crypto key generate rsa modulus 2048
line vty 0 4
 transport input ssh
 login local
```

---
🔗 Relacionado: [[SSH en Cisco]] · [[Configuración básica de equipos Cisco]] · [[Acceso y autenticación segura en Cisco]]
