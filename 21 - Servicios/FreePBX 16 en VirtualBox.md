---
tags:
  - voip
  - freepbx
  - asterisk
  - virtualbox
  - servicios
aliases:
  - FreePBX
  - Instalar FreePBX
  - Extensiones SIP
  - Linphone
---

# FreePBX 16 en VirtualBox

Central telefónica IP **FreePBX 16** (Asterisk 20) instalada desde la ISO de Sangoma en una VM de VirtualBox, con extensiones PJSIP y softphones como clientes. Toma unos 40 a 50 minutos en total.

> [!info] Versión
> FreePBX 16 corre sobre SNG7, basado en CentOS 7, que ya no recibe soporte. **FreePBX 17** se instala con un script sobre Debian 12. Para laboratorio, la ISO 16 sigue siendo lo más rápido.

## VM

| Campo | Valor |
|---|---|
| ISO | `SNG7-PBX16-64bit-2306-1.iso` (≈1,7 GB) desde `downloads.freepbxdistro.org` |
| Tipo / versión | Linux / Red Hat (64-bit) |
| CPU / RAM | 2 núcleos (4 recomendado) / 4 GB |
| Disco | VDI dinámico de 25 GB |
| Red | **Adaptador puente** o **Red NAT** |

> [!warning] NAT simple no deja entrar al panel
> Con el modo "NAT" de VirtualBox la VM queda en `10.0.2.15` y el anfitrión no la alcanza sin reenvío de puertos. Para administrarla y conectar softphones usa adaptador puente, red NAT o un segundo adaptador *solo anfitrión*.

## Instalación

1. Arranca desde la ISO y elige **FreePBX 16 Installation (Asterisk 20) - Recommended**.
2. **Graphical Installation - Output to VGA**.
3. **FreePBX Automatic Installation**. La contraseña de root queda en `SangomaDefaultPassword`.
4. Espera de 15 a 25 minutos hasta **Complete!** y pulsa **Reboot**; desmonta la ISO.
5. La consola muestra la IP. Abre `https://IP/admin` y acepta el certificado autofirmado. Tras arrancar, el panel puede tardar de 3 a 5 minutos.

> [!tip] Si la clave de root no funciona
> Cópiala exacta: `SangomaDefaultPassword`, sin una "d" extra. Cámbiala apenas entres con `passwd root`.

## Configuración inicial (web)

1. **Administrator user:** usuario `admin`, contraseña robusta, correo de notificaciones opcional e identificador del sistema.
2. Actualizaciones automáticas de módulos y de seguridad: activadas.
3. **Locales:** prompts y sistema en español.
4. **Sangoma Smart Firewall:** marca como confiables tu equipo (`/32`) y tu red de laboratorio.

## Extensiones PJSIP

**Applications → Extensions → SIP [chan_pjsip] Extensions → + Quick Create Extension**.

| Campo | Valor de ejemplo |
|---|---|
| User Extension | 100 |
| Display Name | Usuario1 |
| Secret | Generado (anótalo) |

Repite para 101, 102 y siguientes. Datos que necesita cada softphone: extensión, secret, IP del servidor y puerto **5060/UDP**.

## Softphones

| Plataforma | Cliente |
|---|---|
| Linux | Linphone (`sudo apt install linphone-desktop`), Twinkle, `pjsua` |
| Windows | MicroSIP, Zoiper |
| Android / iOS | Linphone, Zoiper |

En Linphone: **Settings → Accounts → Add account → SIP**, con usuario igual a la extensión, el secret, el dominio igual a la IP del servidor y el puerto 5060. El estado debe quedar **Online**; prueba llamando de la 100 a la 101 en ambos sentidos.

## Problemas frecuentes

| Problema | Solución |
|---|---|
| El panel no carga | Revisa el modo de red, espera unos minutos tras el arranque y acepta el certificado |
| El firewall te bloqueó | En consola: `fwconsole firewall trust IP_DE_TU_PC` o, temporalmente, `fwconsole firewall stop` |
| El softphone no registra | Secret, IP del servidor, puerto 5060 y que el softphone esté en una red confiable del firewall |
| Audio en un solo sentido | NAT entre teléfono y servidor: revisa **Settings → Asterisk SIP Settings** (red local y IP externa) y los puertos RTP 10000-20000/UDP |

Recuperar root desde GRUB: pulsa `e`, agrega `rd.break` al final de la línea `linux16`, arranca con **Ctrl+X** y ejecuta:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel      # obligatorio en sistemas con SELinux
exit
```

## Siguientes pasos

| Función | Menú |
|---|---|
| IVR (menú interactivo) | Applications → IVR |
| Colas de llamadas | Applications → Queues |
| Buzón de voz | Applications → Voicemail |
| Troncales SIP externas | Connectivity → Trunks |
| QoS para voz en la red | [[QoS en router y switch con FreePBX]] |

| Componente | Versión |
|---|---|
| FreePBX | 16 (SNG7 2306) |
| Asterisk | 20 LTS |
| Sistema | SNG7 (CentOS 7) |
| Base de datos | MariaDB |
| Canal SIP | PJSIP |

---
🔗 Relacionado: [[PBX (Asterisk - FreePBX) en AWS]] · [[QoS en router y switch con FreePBX]] · [[QoS para voz sin switch administrable]] · [[Capacity Planning y QoS]]
