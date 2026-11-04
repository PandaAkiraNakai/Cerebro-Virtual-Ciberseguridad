---
tags:
  - cisco
  - seguridad
  - ssh
  - acceso-remoto
aliases:
  - SSH Cisco
  - SSH router Cisco
  - SSHv2 Cisco
---

# SSH en dispositivos Cisco

**SSH (Secure Shell)** permite administrar routers y switches de forma remota con la comunicación **cifrada**. Reemplaza a [[Telnet en Cisco IOS|Telnet]], que envía usuario, contraseña y comandos en texto plano.

| Propiedad | Cómo la entrega SSH |
|---|---|
| Autenticación | Contraseña o clave pública |
| Cifrado | Algoritmos simétricos como AES |
| Integridad | Códigos MAC que detectan alteraciones en tránsito |

## Versiones en Cisco IOS

| `show ip ssh` muestra | Significado |
|---|---|
| `version 1.5` | Solo SSHv1: obsoleto e inseguro |
| `version 1.99` | Compatibilidad: acepta v1 y v2. Es lo que queda al generar las claves |
| `version 2.0` | Solo SSHv2: lo recomendado |

- `1.99` no es una versión del protocolo; indica que el equipo acepta ambas.
- SSHv2 exige claves RSA de **768 bits como mínimo**; usa 2048.
- En SSHv2 con clave pública, el cliente **firma** un dato de la sesión con su clave privada y el servidor verifica la firma. El esquema de "desafío que el cliente descifra" es de SSHv1.

## Prerrequisitos

1. Hostname distinto al de fábrica (`Router` o `Switch`).
2. Nombre de dominio.
3. Claves RSA generadas.
4. Usuario local o servidor AAA (ver [[TACACS+ con tac_plus-ng]]).
5. IP alcanzable (interfaz o SVI).

> [!warning] Hostname y dominio primero
> Se usan para nombrar las claves RSA. Si faltan, `crypto key generate rsa` da error.

## Configuración paso a paso

```cisco
enable
configure terminal
 hostname R1
 ip domain-name ejemplo.com
 enable secret Cisco123
 username admin privilege 15 secret Admin123      ! secret, nunca password (tipo 7 es reversible)
 crypto key generate rsa general-keys modulus 2048
 ip ssh version 2                                 ! después de generar las claves
 ip ssh time-out 60                               ! segundos para autenticarse
 ip ssh authentication-retries 3
 line vty 0 4                                     ! en switches existen también 5-15
  transport input ssh                             ! solo SSH, Telnet bloqueado
  login local                                     ! usuarios locales
  exec-timeout 5 0                                ! cierra tras 5 min inactivo
 exit
end
copy running-config startup-config
```

> [!warning] Escribe siempre 2048
> Con solo `crypto key generate rsa` el equipo pregunta el tamaño y propone 512. Con 512 bits SSHv2 no funciona.

IP de gestión según el equipo:

```cisco
! Router: interfaz física
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

! Switch: SVI y gateway
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
ip default-gateway 192.168.1.1
```

Opcional fuera de Packet Tracer: `crypto key encrypt rsa` cifra las claves RSA guardadas.

## Verificación

```cisco
show ip ssh                    ! se espera "SSH Enabled - version 2.0"
show ssh                       ! sesiones activas
show crypto key mypubkey rsa   ! claves generadas
show users                     ! usuarios conectados por VTY
```

- Si aparece `1.99`, falta `ip ssh version 2`.
- Si aparece `SSH Disabled`, faltan las claves RSA.

## Probar desde una PC (Packet Tracer)

En **Desktop > Command Prompt**:

```bash
ssh -l admin 192.168.1.1
```

Con privilegio 15 se entra directo al modo privilegiado; si no, usa `enable`.

## Errores comunes

| Síntoma | Causa | Solución |
|---|---|---|
| `Please define a hostname other than Router` | Hostname por defecto | `hostname R1` |
| `Please define a domain-name first` | Falta dominio | `ip domain-name ejemplo.com` |
| `ip ssh version 2` falla | Claves de 512 bits | Regenerar con `modulus 2048` |
| `SSH Disabled` | No hay claves RSA | `crypto key generate rsa general-keys modulus 2048` |
| `Connection refused` | VTY sin `transport input ssh` o sin IP | Revisa VTY e interfaz |
| `% Login invalid` | Credenciales erradas o falta `login local` | Revisa `username` y VTY |
| `crypto key encrypt rsa` no existe | No soportado en Packet Tracer | Omítelo en PT |

> [!tip] Restringir el acceso
> Combina con una [[ACL estándar]] en las VTY (`access-class ... in`) para permitir SSH solo desde IPs de administración. Ver [[Acceso y autenticación segura en Cisco]].

---
📎 Guía completa: ![[SSH-Router-Cisco.pdf]]

🔗 Relacionado: [[Telnet en Cisco IOS]] · [[Acceso y autenticación segura en Cisco]] · [[Usuarios y niveles de privilegio]] · [[AAA en Cisco]] · [[TACACS+ con tac_plus-ng]] · [[SSH en Linux]]
