---
tags:
  - pentesting
  - esteganografia
  - forense
  - ctf
  - stegseek
aliases:
  - Esteganografía
  - StegSeek
  - Steghide
  - Stego
---

# Esteganografía y análisis de archivos

La **esteganografía** oculta información dentro de otro archivo aparentemente inocente (imagen, audio, documento). En pentesting y sobre todo en **CTF/forense** aparece al revés: hay que **detectar y extraer** datos escondidos en un archivo entregado. Esta nota cubre el flujo de análisis y las herramientas clave, con [[#StegSeek]] como protagonista para romper la protección de Steghide.

> [!warning] Uso ético
> Analiza solo archivos **propios, de un CTF o con autorización**. Ver [[📜 Fuentes y Licencias]].

## Flujo de análisis (de menos a más intrusivo)

```bash
file sospechosa.jpg          # tipo real del archivo (no te fíes de la extensión)
exiftool sospechosa.jpg      # metadatos EXIF: comentarios, GPS, autor, campos raros
strings -n 6 sospechosa.jpg  # cadenas legibles embebidas (flags, rutas, claves)
binwalk sospechosa.jpg       # ¿hay otro archivo dentro? (ZIP, PNG, ejecutable)
binwalk -e sospechosa.jpg    # extraer lo que encuentre
```

| Herramienta | Para qué |
|-------------|----------|
| `file` | identificar el formato real |
| `exiftool` | leer/editar metadatos (EXIF, comentarios) |
| `strings` | volcar texto embebido |
| `binwalk` | detectar y extraer archivos anidados (carving) |
| `zsteg` | datos ocultos en **PNG/BMP** por LSB |
| `steghide` | ocultar/extraer en JPEG/BMP/WAV/AU con contraseña |
| `stegseek` | romper por diccionario la contraseña de Steghide |
| `stegsolve` | inspección visual por planos de bits (GUI, Java) |

## Steghide — ocultar y extraer (con contraseña)

```bash
# Ocultar un archivo dentro de una imagen
steghide embed -cf portada.jpg -ef secreto.txt     # pide passphrase

# Extraer (si conoces la contraseña)
steghide extract -sf portada.jpg

# Ver si hay algo embebido / su tamaño
steghide info portada.jpg
```

Formatos soportados por Steghide: **JPEG, BMP, WAV, AU**.

## StegSeek

**StegSeek** hace un ataque por diccionario de alto rendimiento (~100.000 contraseñas/segundo) contra archivos protegidos con **Steghide**, recupera la contraseña y **extrae el contenido automáticamente**. Es muchísimo más rápido que forzar Steghide a mano.

### Instalación
```bash
# Debian/Ubuntu/Kali (paquete .deb)
wget https://github.com/RickdeJager/stegseek/releases/latest/download/stegseek_0.6-1.deb
sudo dpkg -i stegseek_0.6-1.deb
```

### Uso
```bash
# Sintaxis
stegseek [opciones] <archivo_stego> <diccionario> [<archivo_salida>]

# Ataque por diccionario (lo habitual)
stegseek imagen.jpg /usr/share/wordlists/rockyou.txt

# Modo silencioso + archivo de salida (para scripts)
stegseek imagen.jpg rockyou.txt -q -o extraido.bin

# Comprobar solo si hay datos embebidos, sin crackear
stegseek --seed imagen.jpg
```

| Opción | Función |
|--------|---------|
| `-q`, `--quiet` | salida mínima (scripting) |
| `-o`, `--output` | archivo de salida |
| `-f`, `--force` | sobrescribir salida existente |
| `-v`, `--verbose` | salida detallada |

### Códigos de salida (scripting)
`0` éxito · `1` contraseña no encontrada · `2` argumentos inválidos · `3` archivo corrupto/no soportado · `4` error de E/S.

```bash
stegseek objetivo.jpg rockyou.txt -q -o salida.bin
if [ $? -eq 0 ]; then echo "Encontrada → salida.bin"; else echo "Sin resultado"; fi
```

## zsteg — LSB en PNG/BMP

```bash
zsteg imagen.png        # prueba las combinaciones LSB comunes
zsteg -a imagen.png     # todas las técnicas
zsteg -E b1,rgb,lsb,xy imagen.png > extraido.bin   # extraer un canal concreto
```

## Checklist mental para un reto de stego
1. `file` + `exiftool` + `strings` (lo rápido primero).
2. `binwalk -e` por si hay un archivo dentro de otro.
3. Según formato: `zsteg` (PNG/BMP) o `steghide`/`stegseek` (JPG/WAV).
4. ¿Audio? → espectrograma (Audacity/Sonic Visualiser).
5. ¿Pistas en el enunciado? → úsalas como contraseña del diccionario.

## Relación con otras notas
- Diccionarios → [[Hydra y Medusa]] · [[Fuzzing de Directorios y Subdominios]]
- Práctica tipo CTF → [[Laboratorio DVWA (CTF)]]
- Cracking de contraseñas → [[Hashcat]] · [[John the Ripper]]

---
Fuente adaptada: `pulentoski/Herramientas-de-pentesting` (sección esteganografía / StegSeek). Ver [[📜 Fuentes y Licencias]].
