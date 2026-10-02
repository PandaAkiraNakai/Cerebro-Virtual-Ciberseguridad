---
tags:
  - pentesting
  - cracking
  - john
  - contraseñas
aliases:
  - John the Ripper
  - JtR
  - john
---

# John the Ripper — Cracking offline multiformato

**John the Ripper (JtR)** es un crackeador de contraseñas offline centrado en la CPU y en la enorme variedad de formatos que entiende. Su gran ventaja frente a [[Hashcat]] es el ecosistema de scripts `*2john`, que convierten archivos protegidos (SSH, ZIP, PDF, Office, KeePass…) en un hash que John puede atacar. La versión recomendada es **John the Ripper Jumbo**.

> [!warning] Uso ético
> Solo sobre material **propio o autorizado**. Ver [[📜 Fuentes y Licencias]].

## Flujo de trabajo

1. **Extract** — convertir el archivo original en un hash legible (`*2john`).
2. **Crack** — atacar el hash con diccionario, reglas o fuerza bruta.
3. **Show** — recuperar la contraseña del "pot".

## 1. Extracción de hashes (`*2john`)

| Objetivo | Script | Comando |
|----------|--------|---------|
| Clave privada SSH | `ssh2john` | `ssh2john id_rsa > hash.txt` |
| ZIP | `zip2john` | `zip2john archivo.zip > hash.txt` |
| RAR | `rar2john` | `rar2john archivo.rar > hash.txt` |
| 7-Zip | `7z2john.pl` | `perl 7z2john.pl archivo.7z > hash.txt` |
| PDF | `pdf2john.pl` | `pdf2john.pl doc.pdf > hash.txt` |
| Office | `office2john.py` | `office2john.py doc.docx > hash.txt` |
| KeePass | `keepass2john` | `keepass2john base.kdbx > hash.txt` |
| Linux `/etc/shadow` | `unshadow` | `unshadow /etc/passwd /etc/shadow > hash.txt` |

> [!tip] Encontrar los scripts
> En Kali suelen estar en `/usr/share/john/` o directamente en el `$PATH` (`locate 2john`). Ver [[Proceso de Extracción de Hashes]] para el detalle por formato.

## 2. Cracking

### Diccionario
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Diccionario + reglas (mutaciones)
```bash
john --wordlist=rockyou.txt --rules=Jumbo hash.txt
```

### Forzar el formato cuando John no lo detecta
```bash
john --format=raw-sha256 --wordlist=rockyou.txt hash.txt
john --list=formats | tr ',' '\n' | grep -i sha   # ver formatos disponibles
```

### Modo incremental (fuerza bruta pura)
```bash
john --incremental hash.txt
```

## 3. Ver resultados

```bash
john --show hash.txt              # muestra usuario:contraseña ya crackeados
cat ~/.john/john.pot              # "pot" con todo lo descifrado
```

## Opciones útiles

| Opción | Función |
|--------|---------|
| `--wordlist=` | Diccionario a usar |
| `--rules[=nombre]` | Aplica reglas de mutación |
| `--format=` | Fuerza el tipo de hash |
| `--show` | Muestra lo ya crackeado |
| `--incremental` | Fuerza bruta por caracteres |
| `--fork=4` | Usa 4 procesos (multinúcleo) |
| `--session=nombre` | Nombra la sesión (reanudable con `--restore`) |

## John vs Hashcat

| | John the Ripper | [[Hashcat]] |
|---|---|---|
| Motor | CPU (GPU en Jumbo, parcial) | GPU (muy superior) |
| Fuerte en | formatos raros, scripts `*2john`, comodidad | velocidad bruta, hashes masivos |
| Autodetección | sí, muy buena | manual (`-m`) |

> Regla práctica: usa **John** para extraer/identificar y atacar formatos poco comunes; pásate a **Hashcat** si tienes GPU y muchos hashes.

## Relación con otras notas
- Extracción detallada → [[Proceso de Extracción de Hashes]]
- Cracking por GPU → [[Hashcat]]
- Fuerza bruta online → [[Hydra y Medusa]]
- Hashes de Windows / AD → [[Escalada de Privilegios Windows]] · [[Active Directory — Ataques]]

---
Fuente adaptada: `pulentoski/Herramientas-de-pentesting`. Ver [[📜 Fuentes y Licencias]].
