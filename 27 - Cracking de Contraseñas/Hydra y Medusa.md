---
tags:
  - pentesting
  - cracking
  - fuerza-bruta
  - hydra
  - medusa
aliases:
  - Hydra
  - Medusa
  - Fuerza bruta online
---

# Hydra y Medusa — Fuerza bruta online contra servicios

A diferencia de [[Hashcat]]/[[John the Ripper]] (que atacan hashes **offline**), **Hydra** y **Medusa** hacen fuerza bruta **online**: prueban combinaciones de usuario/contraseña directamente contra un servicio en red (SSH, FTP, RDP, SMB, formularios web…). Son rápidas y multiprotocolo, pero **ruidosas**: generan tráfico y pueden disparar bloqueos de cuenta o alertas del IDS ([[Snort]]/[[Suricata]]).

> [!warning] Uso ético y ruido
> Solo contra servicios **propios o autorizados**. La fuerza bruta online deja huella en logs y puede bloquear cuentas legítimas. Ver [[📜 Fuentes y Licencias]].

---

## Hydra

### Sintaxis
```bash
hydra -l USUARIO -P DICCIONARIO -t HILOS PROTOCOLO://IP
```

| Opción | Función |
|--------|---------|
| `-l` / `-L` | un usuario / lista de usuarios |
| `-p` / `-P` | una contraseña / diccionario |
| `-t` | nº de hilos (6–16 razonable; más = más ruido) |
| `-V` | modo verboso (muestra cada intento) |
| `-f` | detener al encontrar la primera válida |
| `-s` | puerto no estándar |

### Servicios de red
```bash
# SSH
hydra -l admin -P rockyou.txt -t 6 ssh://10.10.10.5

# FTP
hydra -l usuario -P claves.txt -t 6 ftp://10.10.10.5

# RDP
hydra -l administrator -P rockyou.txt rdp://10.10.10.5
```

### Formularios web
La clave es indicar la ruta, los parámetros (con `^USER^` y `^PASS^`) y el **mensaje de error** que devuelve la página al fallar.

```bash
# POST (login típico)
hydra -l admin -P rockyou.txt 10.10.10.5 http-post-form \
  "/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid" -V

# GET con autenticación básica
hydra -l bob -P rockyou.txt -f 10.10.10.5 http-get /protected
```

> [!tip] Capturar los parámetros del formulario
> Usa el proxy de Burp o las DevTools del navegador para ver la petición real (ruta, nombres de campo y el texto exacto del error). Un error mal escrito = falso negativo en todos los intentos.

---

## Medusa

Misma idea que Hydra, con sintaxis propia. Útil como alternativa cuando un módulo de Hydra falla.

| Opción | Función |
|--------|---------|
| `-h` / `-H` | host / archivo de hosts |
| `-u` / `-U` | usuario / archivo de usuarios |
| `-p` / `-P` | contraseña / archivo de contraseñas |
| `-M` | módulo/protocolo (`ssh`, `ftp`, `smbnt`, `rdp`, `http`) |
| `-t` | nº de hilos |
| `-f` | detener al primer acierto |
| `-e ns` | probar contraseña vacía (`n`) e igual al usuario (`s`) |

```bash
# SSH
medusa -h 192.168.1.10 -u usuario -P claves.txt -M ssh -t 4

# SMB (Samba / Windows)
medusa -h 192.168.1.10 -U usuarios.txt -P claves.txt -M smbnt -t 4

# RDP
medusa -h 10.10.10.5 -u administrator -P rockyou.txt -M rdp

# HTTP con ruta protegida (auth básica)
medusa -h 192.168.1.10 -U usuarios.txt -P claves.txt -M http -m DIR:/ruta/protegida -t 4
```

---

## Preparar diccionarios

```bash
# Invertir rockyou (a veces las comunes están al final)
tac /usr/share/wordlists/rockyou.txt > rockyou_rev.txt

# Quitar espacios en blanco
sed -i 's/ //g' rockyou_rev.txt
```

> [!info] ¿Hydra o Medusa?
> Hydra tiene más módulos y mejor soporte para formularios web; Medusa paraleliza mejor contra **muchos hosts** a la vez. Para SMB/AD en entornos Windows, considera además [[CrackMapExec - NetExec]], que además enumera y hace spraying de forma más sigilosa.

## Relación con otras notas
- Fuerza bruta web y límites de tasa → [[Fuerza bruta y rate limiting]]
- Cracking offline (hashes ya obtenidos) → [[Hashcat]] · [[John the Ripper]]
- Spraying/credenciales en AD → [[CrackMapExec - NetExec]]
- Ya aparecía en → [[Pentesting en Redes]]

---
Fuente adaptada: `pulentoski/Herramientas-de-pentesting`. Ver [[📜 Fuentes y Licencias]].
