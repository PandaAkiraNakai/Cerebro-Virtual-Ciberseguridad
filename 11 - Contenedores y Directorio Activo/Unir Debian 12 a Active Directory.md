---
tags:
  - linux
  - active-directory
  - debian
  - autenticacion
aliases:
  - Debian 12 en Active Directory
  - Unir Linux a un dominio AD
---

# Unir Debian 12 a Active Directory

**Guía para integrar un equipo Debian 12 ("bookworm") a un dominio de Active Directory (AD). Esto permite que un usuario se autentique con sus credenciales del dominio y acceda de forma automática a los recursos compartidos de la red.**

## Requisitos previos

- Credenciales de administrador del dominio.
- Sistema Debian 12 ("bookworm") instalado en el equipo.

## Herramientas y paquetes necesarios

Para la unión al dominio y la configuración de autenticación y autorización se usan **SSSD** y **pam_mount**:

| Herramienta | Función |
|---|---|
| **SSSD** (System Security Services Daemon) | Gestiona la conectividad con el dominio, permitiendo que los servicios comunes de GNU/Linux usen mecanismos de autenticación y autorización basados en AD. |
| **pam_mount** | Monta automáticamente las particiones o recursos de red cuando el usuario inicia sesión. |

### Instalación de paquetes

```bash
sudo apt install realmd sssd samba-common krb5-user adcli \
  libsss-sudo sssd-tools libsasl2-modules-ldap packagekit libpam-mount
```

## Pasos para unirse al dominio

### 1. Configuración de Kerberos

Edita el archivo de configuración de Kerberos con tu dominio:

```bash
sudo nano /etc/krb5.conf
```

En la sección `[libdefaults]`, añade el nombre de tu dominio:

```ini
[libdefaults]
default_realm = MIDOMINIO.LOCAL
```

### 2. Unir Debian al dominio

Usa el comando `realm join`. Se necesita un usuario administrador del dominio:

```bash
sudo realm join --user=ADMIN_USER MIDOMINIO.LOCAL -v
```

### 3. Verificar la conexión al dominio

Para asegurarte de que el equipo se unió correctamente:

```bash
sudo realm list
```

Verás información sobre el dominio: tipo de servidor, software cliente utilizado y la política de inicio de sesión.

## Configuración de SSSD

Edita `/etc/sssd/sssd.conf` para establecer las configuraciones del dominio:

```bash
sudo nano /etc/sssd/sssd.conf
```

Añade la siguiente configuración (ajustando `midominio.local` según corresponda):

```ini
[sssd]
domains = midominio.local
config_file_version = 2
default_domain_suffix = midominio.local

[domain/midominio.local]
id_provider = ad
access_provider = ad
krb5_realm = MIDOMINIO.LOCAL
default_shell = /bin/bash
fallback_homedir = /home/%u@%d
cache_credentials = True
```

Reinicia el servicio SSSD para aplicar los cambios:

```bash
sudo systemctl restart sssd
```

> [!warning] Permisos de sssd.conf
> El archivo `/etc/sssd/sssd.conf` debe pertenecer a root y tener permisos `600`. Si no, SSSD se niega a arrancar: `sudo chown root:root /etc/sssd/sssd.conf && sudo chmod 600 /etc/sssd/sssd.conf`.

## Configuración de PAM (autenticación y recursos de red)

### Activar módulos PAM

Usa `pam-auth-update` para activar los módulos necesarios:

```bash
sudo pam-auth-update
```

Marca las siguientes opciones:

- **SSS authentication**
- **Mount volumes for user**
- **Create home directory on login**

### Configuración de pam_mount

Edita `/etc/security/pam_mount.conf.xml` para que `pam_mount` monte automáticamente los recursos compartidos del dominio:

```xml
<pam_mount>
  <debug enable="0" />
  <mntoptions deny="suid,dev,exec" />
  <mntoptions require="nosuid,nodev,noexec" />
  <volume fstype="cifs" sgrp="domain users"
    server="miservidor.midominio.local" path="recurso"
    mountpoint="~/Network Drives/Recurso"
    options="vers=3.0,sec=krb5i,cruid=%(USERUID),nodev,nosuid,noexec,rw" />
  <mkmountpoint enable="1" remove="true" />
</pam_mount>
```

## Configuración de Samba

Para finalizar, ajusta Samba para que Debian reconozca el dominio correctamente:

```bash
sudo nano /etc/samba/smb.conf
```

Añade o ajusta las siguientes líneas:

```ini
[global]
  workgroup = MIDOMINIO
  realm = MIDOMINIO.LOCAL
  encrypt passwords = yes
  client protection = encrypt
```

## Pruebas

**Verificar el estado del sistema** — confirma que el estado no esté degradado y que los servicios de SSSD estén en ejecución:

```bash
systemctl status
```

**Estado del dominio** — verifica que el dominio esté en línea y que los servidores se detecten correctamente:

```bash
sssctl domain-status midominio.local
```

**Iniciar sesión con credenciales de dominio** — cierra sesión y vuelve a iniciar con un usuario de dominio para confirmar que Debian acepta las credenciales de AD.

**Validar montaje de recursos de red** — al iniciar sesión, revisa que las carpetas compartidas se monten en la ubicación especificada (`~/Network Drives/Recurso`).

## Troubleshooting común

| Problema | Dónde mirar |
|---|---|
| `realm join` falla | Verificar resolución DNS del dominio y sincronización horaria (Kerberos es sensible al reloj) |
| Usuario de dominio no puede entrar | `sssctl domain-status midominio.local` y `journalctl -u sssd` |
| El home no se crea al iniciar sesión | Revisar que esté marcado "Create home directory on login" en `pam-auth-update` |
| El recurso de red no se monta | Revisar `pam_mount.conf.xml` y los logs con `journalctl` |
| Hora desfasada rompe Kerberos | `timedatectl` — sincronizar NTP con el dominio |

---
📎 Guías: ![[Unir-Debian-12-Active-Directory.pdf]]

🔗 Relacionado: [[05 - Usuarios, grupos y permisos]] · [[06 - Servicios con systemd]] · [[08 - Seguridad del servidor]]
