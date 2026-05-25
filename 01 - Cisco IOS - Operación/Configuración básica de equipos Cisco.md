---
tags:
  - cisco
  - fundamentos
  - operación
aliases:
  - Config básica Cisco
  - Hostname y contraseñas
---

# Configuración básica de equipos Cisco

Aspectos fundamentales para dejar operativo un switch o router Cisco: nombre, contraseñas, líneas de acceso y cifrado.

## Cambiar el nombre del equipo

```cisco
Switch# configure terminal
Switch(config)# hostname Sw-Floor-1
```

## Contraseñas de acceso

**Consola** (acceso físico):

```cisco
Sw-Floor-1(config)# line console 0
Sw-Floor-1(config-line)# password cisco
Sw-Floor-1(config-line)# login
Sw-Floor-1(config-line)# end
```

**Modo privilegiado EXEC** (cifrada):

```cisco
Sw-Floor-1(config)# enable secret class
```

**Líneas VTY** (acceso remoto por Telnet/[[SSH en Cisco|SSH]]):

```cisco
Sw-Floor-1(config)# line vty 0 15
Sw-Floor-1(config-line)# password cisco
Sw-Floor-1(config-line)# login
Sw-Floor-1(config-line)# end
```

## Cifrar todas las contraseñas

```cisco
Sw-Floor-1(config)# service password-encryption
```

> [!tip] Guardar la configuración
> Tras cualquier cambio: `copy running-config startup-config`

---
📎 Guía base: ![[Config-Basica-Estatica-DHCP.pdf]]

🔗 Relacionado: [[Direcciones IP estáticas y rutas estáticas]] · [[Servicio DHCP en Cisco]] · [[Acceso y autenticación segura en Cisco]]
