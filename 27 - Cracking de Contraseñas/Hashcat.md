---
tags:
  - pentesting
  - cracking
  - hashcat
  - contraseñas
aliases:
  - Hashcat
  - Cracking con GPU
---

# Hashcat — Cracking de Hashes acelerado por GPU

**Hashcat** es el crackeador de contraseñas offline más rápido: aprovecha la **GPU** para probar miles de millones de candidatas por segundo contra un hash. Soporta cientos de algoritmos (MD5, SHA-1/256/512, bcrypt, NTLM, Kerberos, WPA, etc.) y varios modos de ataque. Es la herramienta de referencia para evaluar la resistencia de contraseñas una vez que se ha obtenido el hash (ver [[Proceso de Extracción de Hashes]]).

> [!warning] Uso ético
> Solo sobre hashes **propios o con autorización explícita**. Ver [[📜 Fuentes y Licencias]].

## Sintaxis general

```bash
hashcat -m <modo_hash> -a <modo_ataque> <archivo_hash> <diccionario|máscara>
```

- `-m` — tipo de hash (ver tabla).
- `-a` — modo de ataque (0 diccionario, 3 fuerza bruta/máscara, 6 híbrido).
- `<archivo_hash>` — fichero con uno o varios hashes (uno por línea).

## Modos de hash (`-m`) más usados

| `-m` | Algoritmo | Dónde aparece |
|------|-----------|---------------|
| `0` | MD5 | hashes genéricos, CTFs |
| `100` | SHA1 | genérico |
| `1400` | SHA-256 | genérico |
| `1700` | SHA-512 | genérico |
| `1800` | sha512crypt `$6$` | `/etc/shadow` Linux moderno |
| `500` | md5crypt `$1$` | `/etc/shadow` antiguo |
| `3200` | bcrypt `$2*$` | apps web, Laravel |
| `1000` | NTLM | hashes de Windows (SAM/NTDS) |
| `5600` | NetNTLMv2 | capturas con Responder |
| `13100` | Kerberos TGS (Kerberoasting) | ver [[Active Directory — Ataques]] |
| `22000` | WPA-PBKDF2-PMKID+EAPOL | handshakes WiFi |
| `16500` | JWT (HS256) | ver [[JWT (JSON Web Token)]] |

> [!tip] Identificar el tipo
> Si no sabes el algoritmo: `hashid '<hash>'` o `hashcat --identify hash.txt`. En CTF, el prefijo (`$6$`, `$2y$`, `$krb5tgs$`) ya lo delata.

## Modos de ataque (`-a`)

### 0 — Diccionario (el más común)
```bash
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

### 0 + reglas (mutación de palabras)
Las reglas transforman cada palabra del diccionario (mayúsculas, `password`→`P@ssw0rd!`, etc.):
```bash
hashcat -m 0 -a 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

### 3 — Máscara / fuerza bruta
Juegos de caracteres: `?l` minús, `?u` mayús, `?d` dígito, `?s` símbolo, `?a` todo.
```bash
# 8 caracteres: mayúscula + 6 minúsculas + 2 dígitos
hashcat -m 0 -a 3 hashes.txt '?u?l?l?l?l?l?l?d?d'
```

### 6 — Híbrido (diccionario + máscara)
```bash
# palabra del diccionario seguida de 3 dígitos
hashcat -m 0 -a 6 hashes.txt rockyou.txt '?d?d?d'
```

## Flujo de trabajo típico

```bash
# 1. Lanzar el ataque
hashcat -m 1800 -a 0 shadow.hash rockyou.txt -O -w 3

# 2. Ver las contraseñas ya crackeadas (potfile)
hashcat -m 1800 shadow.hash --show

# 3. Reanudar / ver estado de una sesión
hashcat --restore
```

## Opciones útiles

| Opción | Función |
|--------|---------|
| `--show` | Muestra hashes ya descifrados (los guarda en `~/.hashcat/hashcat.potfile`) |
| `-O` | Kernel optimizado (más rápido, limita longitud de contraseña) |
| `-w 3` | Perfil de carga (1 bajo … 4 pesado) |
| `--force` | Ignora avisos (útil en VM sin GPU) |
| `-o cracked.txt` | Guardar resultados en archivo |
| `--username` | El archivo tiene formato `usuario:hash` |
| `--status --status-timer=10` | Estado automático cada 10 s |
| `-i` | Incremento de longitud en máscara |

> [!info] Benchmark
> `hashcat -b` mide la velocidad de tu equipo por algoritmo. Sin GPU, Hashcat funciona por CPU pero mucho más lento; para CPU puro suele ir mejor [[John the Ripper]].

## Relación con otras notas
- Extracción previa del hash → [[Proceso de Extracción de Hashes]]
- Alternativa en CPU y con formatos exóticos → [[John the Ripper]]
- Fuerza bruta **online** (contra servicios) → [[Hydra y Medusa]]
- Diccionarios → [[Fuzzing de Directorios y Subdominios]]

---
Fuente adaptada: `pulentoski/Herramientas-de-pentesting`. Ver [[📜 Fuentes y Licencias]].
