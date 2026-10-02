---
tags:
  - pentesting
  - cracking
  - hashes
  - contraseñas
aliases:
  - Proceso de Extracción de Hashes
  - 2john
  - Extraer hashes
---

# Proceso de Extracción de Hashes (`*2john`)

Antes de crackear hay que **obtener el hash**: la representación cifrada de la contraseña que protege un archivo o una cuenta. Los crackeadores ([[John the Ripper]], [[Hashcat]]) no leen archivos binarios directamente, así que primero se convierte el objetivo en una línea de texto (el hash) con un extractor específico.

> [!info] ¿Por qué extraer el hash?
> - **Ataque offline:** se trabaja en la máquina propia, sin tocar el objetivo → sin bloqueos de cuenta ni alertas.
> - **Seguridad en reposo:** las apps nunca guardan la contraseña en claro, solo su hash.
> - **Compatibilidad:** transforma formatos complejos (KeePass, Office) en un formato estándar que John/Hashcat entienden.

> [!warning] Uso ético
> Extraer hashes de sistemas o archivos **ajenos sin permiso es ilegal**. Ver [[📜 Fuentes y Licencias]].

## Tres orígenes de hashes

1. **Archivos locales** — ZIP, RAR, PDF, Office, KeePass descargados del objetivo.
2. **Sistema operativo** — cuentas de usuario (`/etc/shadow` en Linux, base `SAM`/`NTDS.dit` en Windows).
3. **Credenciales de red** — capturadas en tránsito (NTLM, Kerberos) con Responder u otras.

## Checklist de extractores

| Categoría | Objetivo | Herramienta / script |
|-----------|----------|----------------------|
| Linux local | usuarios (`shadow`) | `unshadow /etc/passwd /etc/shadow` |
| Windows local | base SAM / NTDS | `samdump2`, `secretsdump.py` (Impacket) |
| Acceso remoto | claves privadas (RSA/EdDSA) | `ssh2john id_rsa` |
| Comprimidos | ZIP, RAR, 7z | `zip2john`, `rar2john`, `7z2john.pl` |
| Ofimática | PDF, Office | `pdf2john.pl`, `office2john.py` |
| Gestores | KeePass | `keepass2john base.kdbx` |
| WiFi | handshake WPA/WPA2 | `hcxpcapngtool` (luego Hashcat `-m 22000`) |
| Certificados | PFX / P12 | `pfx2john` |

## Casos de uso en un pentest

| Situación | Extractor | Resultado |
|-----------|-----------|-----------|
| Acceso inicial | `ssh2john` | romper una clave SSH y entrar al servidor |
| Movimiento lateral | `keepass2john` | abrir el gestor de contraseñas de un empleado |
| Exfiltración | `pdf2john.pl` | leer documentos confidenciales protegidos |
| Escalada de privilegios | `unshadow` | combinar `passwd`+`shadow` y romper la clave de root |

## Ejemplo completo (clave SSH)

```bash
# 1. Extraer el hash de la clave privada
ssh2john id_rsa > id_rsa.hash

# 2. Crackear con John
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash

# 3. Ver la passphrase encontrada
john --show id_rsa.hash
```

> [!tip] Regla de oro
> **Sin extracción de hash, no hay cracking.** Cada formato tiene su extractor; identifícalo antes de lanzar el diccionario.

## Relación con otras notas
- Crackear el hash → [[John the Ripper]] · [[Hashcat]]
- Hashes de Windows / AD → [[Escalada de Privilegios Windows]] · [[Active Directory — Ataques]]
- Volcado remoto de credenciales SMB → [[CrackMapExec - NetExec]]

---
Fuente adaptada: `pulentoski/Herramientas-de-pentesting`. Ver [[📜 Fuentes y Licencias]].
