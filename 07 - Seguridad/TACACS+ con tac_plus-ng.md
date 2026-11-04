---
tags:
  - cisco
  - seguridad
  - aaa
  - tacacs
  - linux
aliases:
  - TACACS+
  - tac_plus-ng
  - Servidor TACACS
  - AAA centralizado
---

# TACACS+ con tac_plus-ng

Servidor **TACACS+** (RFC 8907) con **tac_plus-ng**, el fork moderno y mantenido de *event-driven-servers*, para autenticar, autorizar y registrar el acceso **SSHv2** a routers Cisco. Probado en Ubuntu 24.04 LTS y Debian 13 (Trixie) con IOS 15.9; también sirve con IOS-XE.

## AAA en TACACS+

| Función | Pregunta | En TACACS+ |
|---|---|---|
| Authentication | ¿Quién eres? | Usuario y contraseña |
| Authorization | ¿Qué puedes hacer? | Nivel de privilegio y comandos permitidos |
| Accounting | ¿Qué hiciste? | Registro de sesiones y comandos |

TACACS+ separa las tres funciones: puedes autenticar con él y además controlar y registrar cada comando.

| Aspecto | TACACS+ | RADIUS |
|---|---|---|
| Transporte | TCP 49 | UDP 1812/1813 |
| Cifrado | Payload completo | Solo la contraseña |
| AAA | Separado | Authn + Authz juntos |
| Accounting por comando | Sí | Básico |
| Uso típico | Administración de equipos de red | VPN, Wi-Fi, 802.1X |

> [!tip] Regla práctica
> TACACS+ para administrar equipos de red; RADIUS para el acceso de usuarios a la red.

## Instalación (Ubuntu 24.04 o Debian 13)

> [!warning] DNS primero
> En CML y VirtualBox, `systemd-resolved` puede quedar apuntando solo a `127.0.0.53`. Síntoma: `Temporary failure resolving 'archive.ubuntu.com'`.

```bash
echo "nameserver 1.1.1.1" > /etc/resolv.conf

apt update
apt install -y build-essential libssl-dev libpam0g-dev git perl \
  libpcre2-dev libcurl4-openssl-dev libldap2-dev libradcli-dev zlib1g-dev libc-ares-dev

cd /tmp
git clone https://github.com/MarcJHuber/event-driven-servers.git
cd event-driven-servers
./configure --minimum
make && make install

which tac_plus-ng         # /usr/local/sbin/tac_plus-ng
tac_plus-ng -v

mkdir -p /etc/tac_plus-ng /var/log/tac_plus-ng
```

Si el usuario `nobody:nobody` no existe en tu distribución, los logs fallan por permisos. En laboratorio se resuelve con `chmod 777 /var/log/tac_plus-ng`; en producción crea un usuario de servicio dueño del directorio.

| Ruta | Contenido |
|---|---|
| `/etc/tac_plus-ng/tac_plus-ng.cfg` | Usuarios, claves y perfiles |
| `/usr/local/sbin/tac_plus-ng` | Binario |
| `/var/log/tac_plus-ng/` | Logs |

## Configuración mínima del servidor

```text
id = spawnd {
    background = no
    listen { port = 49 }          # el ejemplo oficial usa 4949; Cisco espera 49
}

id = tac_plus-ng {
    host world {
        address = 0.0.0.0/0
        key = demo                # clave compartida, idéntica en el router
    }
    profile admin {
        script { if (service == shell) { permit } }
    }
    user test {
        password login = clear testpass
        password pap = login
        profile = admin
    }
}
```

| Bloque | Función |
|---|---|
| `spawnd` | Proceso que escucha en TCP 49 |
| `host` | Qué equipos pueden consultar y con qué clave |
| `profile` | Qué se le permite al usuario |
| `user` | Credenciales y perfil asignado |

```bash
tac_plus-ng -P /etc/tac_plus-ng/tac_plus-ng.cfg     # validar: sin salida = OK
tac_plus-ng -f /etc/tac_plus-ng/tac_plus-ng.cfg &   # iniciar a mano
ss -tlnp | grep :49
```

> [!warning] Clave compartida
> TACACS+ cifra el payload completo con la clave. Si no es **idéntica** en servidor y router, la autenticación falla sin mensajes útiles. Es la causa del 90 % de las fallas.

## Router Cisco

```cisco
configure terminal
 hostname RT-1
 ip domain-name lab.local
 enable secret cisco123
 username admin privilege 15 secret Admin123        ! respaldo local obligatorio
 crypto key generate rsa general-keys modulus 2048
 ip ssh version 2
 ip ssh time-out 60
 ip ssh authentication-retries 3
 aaa new-model
 aaa authentication login TACACS_LOGIN group tacacs+ local
 tacacs server TACACS-SERVER
  address ipv4 10.10.10.2
  key demo
 exit
 line vty 0 4
  login authentication TACACS_LOGIN
  transport input ssh
  exec-timeout 5 0
 line con 0
  no login                                          ! acceso de emergencia
end
write memory
```

- El método `local` usa la base de `username`, **no** el `enable secret`. Sin usuario local, si TACACS+ cae nadie entra por SSH.
- `no login` en consola es la puerta de emergencia; su protección es física (sala y rack con llave).
- El detalle de SSH está en [[SSH en Cisco]] y el de listas de métodos en [[AAA en Cisco]].

### Cómo fluye un login

1. El usuario abre SSH (TCP 22): Diffie-Hellman, clave RSA del router y túnel cifrado.
2. Envía usuario y contraseña dentro del túnel.
3. El router aplica la lista `TACACS_LOGIN`: primero `group tacacs+`.
4. Consulta al servidor por TCP 49 con el payload cifrado con la clave compartida.
5. El servidor responde **PASS** o **FAIL**.

| Resultado | Qué pasa |
|---|---|
| PASS | Entra (`RT-1>`) |
| FAIL | `Authentication failed`; **no** pasa al método local |
| TIMEOUT | Recién ahí prueba el usuario local |

Hay dos capas de cifrado: SSH protege usuario ↔ router y la clave TACACS+ protege router ↔ servidor. El servidor nunca dice si el usuario no existe o si la contraseña está mal, para no revelar usuarios válidos.

## Verificación

```cisco
show ip ssh                     ! SSH Enabled - version 2.0
show run | include aaa
show run | section tacacs
show tacacs                     ! contadores y estado del servidor
```

```bash
ssh test@10.10.10.1             # login con TACACS+
killall tac_plus-ng
ssh admin@10.10.10.1            # fallback local
telnet 10.10.10.1               # debe rechazarse
```

## Administración

### Usuarios con contraseña cifrada

```bash
openssl passwd -6 'ClaveDeAna'          # genera $6$...
grep -E "^\s*user " /etc/tac_plus-ng/tac_plus-ng.cfg   # listar usuarios
```

```text
    user ana {
        password login = crypt $6$Xy...$Ab...
        password pap = login
        profile = admin
    }
```

Usa solo `a-z A-Z 0-9 _ -` en contraseñas en texto claro: caracteres como `@`, `#` o `!` rompen el login. El bloque `user` debe quedar **dentro** de `id = tac_plus-ng { … }`; si lo agregas con `>>` al final del archivo queda fuera.

### Servicio systemd

```ini
# /etc/systemd/system/tac_plus-ng.service
[Unit]
Description=TACACS+ Server (tac_plus-ng)
After=network-online.target
Wants=network-online.target

[Service]
ExecStartPre=/usr/local/sbin/tac_plus-ng -P /etc/tac_plus-ng/tac_plus-ng.cfg
ExecStart=/usr/local/sbin/tac_plus-ng -f /etc/tac_plus-ng/tac_plus-ng.cfg
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
killall tac_plus-ng
systemctl daemon-reload
systemctl enable --now tac_plus-ng
journalctl -u tac_plus-ng -f
```

`ExecStartPre` valida la configuración: con un error, el servicio no arranca roto.

### Logs separados

```text
    log authc { destination = /var/log/tac_plus-ng/authc.log }
    log authz { destination = /var/log/tac_plus-ng/authz.log }
    log acct  { destination = /var/log/tac_plus-ng/acct.log }
    authentication log = authc
    authorization log = authz
    accounting log = acct
```

| Log | Registra |
|---|---|
| `authc.log` | Logins exitosos y fallidos |
| `authz.log` | Decisiones de permitir o denegar |
| `acct.log` | Inicio y fin de sesión y comandos |

Si no se escriben logs, depura en primer plano con `tac_plus-ng -f /etc/tac_plus-ng/tac_plus-ng.cfg -d 8`.

### Privilegio por perfil

```text
    profile admin {
        script { if (service == shell) {
            if (cmd == "") { set priv-lvl = 15  permit }
            permit } }
    }
    profile soporte {
        script { if (service == shell) {
            if (cmd == "") { set priv-lvl = 1  permit }
            permit } }
    }
```

```cisco
aaa authorization exec default group tacacs+ local
```

Resultado: `admin` entra directo a `RT-1#` y `soporte` a `RT-1>`. Compruébalo con `show privilege`.

### Autorización por comando (solo lectura)

```text
    profile readonly {
        script { if (service == shell) {
            if (cmd == "") { set priv-lvl = 15  permit }
            if (cmd =~ /^show /) { permit }
            if (cmd =~ /^ping /) { permit }
            if (cmd =~ /^traceroute /) { permit }
            if (cmd =~ /^exit/) { permit }
            deny } }
    }
```

```cisco
aaa authorization commands 1 default group tacacs+ local
aaa authorization commands 15 default group tacacs+ local
```

> [!danger] Ten la consola a mano
> Un error en el perfil puede bloquear todos los comandos por SSH. Por defecto la consola no usa autorización de comandos.

```text
RT-1# show ip interface brief     → permitido
RT-1# configure terminal          → Command authorization failed.
```

### Grupos y dispositivos por red

```text
    group netadmins
    group auditores
    user ana     { password login = crypt $6$...  password pap = login  member = netadmins }
    user auditor { password login = crypt $6$...  password pap = login  member = auditores }
    ruleset { rule { script {
        if (member == netadmins) { profile = admin permit }
        if (member == auditores) { profile = readonly permit }
        deny } } }

    host core   { address = 172.16.0.0/24  key = ClaveCore }
    host acceso { address = 10.0.0.0/8     key = ClaveAcceso }
```

Las consultas desde IP no declaradas se ignoran, y cada segmento usa su propia clave.

### Redundancia de servidores

```cisco
tacacs server TACACS-1
 address ipv4 172.16.99.10
 key ClaveCore
tacacs server TACACS-2
 address ipv4 172.16.99.11
 key ClaveCore
aaa group server tacacs+ TACACS-GROUP
 server name TACACS-1
 server name TACACS-2
aaa authentication login TACACS_LOGIN group TACACS-GROUP local
```

### Accounting y auditoría

```cisco
aaa accounting exec default start-stop group tacacs+
aaa accounting commands 1 default start-stop group tacacs+
aaa accounting commands 15 default start-stop group tacacs+
```

```bash
grep -i fail /var/log/tac_plus-ng/authc.log | sort | uniq -c     # fallidos
grep ana /var/log/tac_plus-ng/acct.log | grep cmd                # comandos de un usuario
grep "$(date +%Y-%m-%d)" /var/log/tac_plus-ng/acct.log           # actividad de hoy
```

```cisco
show users
debug tacacs
debug aaa authentication
undebug all
```

Los logs se pueden enviar a [[Splunk Enterprise]] para tableros de logins por hora, intentos fallidos y comandos de configuración.

## Troubleshooting

| Síntoma | Causa | Solución |
|---|---|---|
| `no matching key exchange method found` | OpenSSH 9.x desactiva SHA-1, que usa IOS 15.9 | Habilitarlo solo para ese host (abajo) |
| `Connection refused` en el 22 | SSH apagado en el router | Ver [[SSH en Cisco]] |
| `Authentication failed` | Clave compartida distinta | Compara `grep "key = "` con `show run \| section tacacs` |
| El local funciona y TACACS+ no | El servidor no responde y entra el fallback | `ping`, `ps aux`, `ss -tlnp`, `show tacacs` |
| Contraseña correcta pero falla | Caracteres especiales | Solo alfanuméricos, `_` y `-` |
| Consola bloqueada | AAA en consola sin usuario local | En CML: Stop → Wipe → Start y `no login` en consola |
| Enable no pasa por TACACS+ | Limitación de este montaje | Usa `enable secret` local |

```text
# ~/.ssh/config — excepción solo para el equipo legado
Host 10.10.10.1
    KexAlgorithms +diffie-hellman-group14-sha1
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
```

Orden de depuración: ¿responde SSH? → ¿clave idéntica? → ¿existe el usuario? → ¿responde el servidor? → ¿caracteres especiales?

Flujo para cualquier cambio: editar `.cfg` → validar con `-P` → reiniciar → probar login → revisar logs.

## Ejercicios

1. Crea dos usuarios, cambia la contraseña de uno y elimina al otro.
2. Pasa todas las contraseñas a `crypt` y deja el servicio en systemd; reinicia el servidor y comprueba que levanta solo.
3. Haz que un usuario entre con nivel 15 y otro con nivel 1.
4. Crea el perfil `readonly` y verifica que no permite `configure terminal`.
5. Activa accounting de comandos, cambia el hostname de un router y encuentra el comando en `acct.log`: quién, a qué hora y desde qué IP.
6. Captura con [[Wireshark]] los tramos TCP 22 y TCP 49. ¿Qué se ve en cada uno? ¿Qué cambiaría con Telnet?

## Referencias

- RFC 8907 (TACACS+): <https://www.rfc-editor.org/rfc/rfc8907>
- tac_plus-ng: <https://github.com/MarcJHuber/event-driven-servers>

---
🔗 Relacionado: [[AAA en Cisco]] · [[SSH en Cisco]] · [[Usuarios y niveles de privilegio]] · [[Acceso y autenticación segura en Cisco]] · [[06 - Servicios con systemd]] · [[Splunk Enterprise]]
