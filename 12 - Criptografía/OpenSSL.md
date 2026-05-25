---
tags:
  - criptografia
  - openssl
  - tls
aliases:
  - OpenSSL
  - SSL/TLS
---

# OpenSSL

**OpenSSL** — biblioteca de software de código abierto con herramientas para implementar protocolos de seguridad, como el cifrado de datos y la autenticación. Se basa en **SSL** y su sucesor **TLS**, esenciales para asegurar las comunicaciones en red.

## ¿Para qué sirve?

- **Cifrado y descifrado de datos:** proteger la confidencialidad de datos en reposo o en tránsito.
- **Generación y gestión de certificados digitales:** autenticar identidades y habilitar HTTPS en sitios web.
- **Creación de claves criptográficas:** pares de clave pública/privada para cifrado asimétrico.
- **Cálculo de hashes criptográficos:** verificar la integridad de archivos mediante huellas digitales.
- **Implementación de protocolos de seguridad:** configurar servidores para usar HTTPS.

## ¿Dónde se utiliza?

| Contexto | Uso |
|---|---|
| Sitios web seguros (HTTPS) | Certificados SSL/TLS en Apache, Nginx o la nube. |
| Correo electrónico seguro | Cifrar correos y autenticar en servidores que soportan SSL/TLS. |
| Aplicaciones de red | Asegurar comunicación entre apps sobre redes inseguras. |
| Almacenamiento | Cifrar archivos y discos para proteger datos en reposo. |
| VPN | Túneles cifrados para el tráfico de red. |
| Desarrollo de software | Integrar cifrado y autenticación en aplicaciones propias. |

## Instalación

En Kali Linux suele venir preinstalado. Para asegurar la última versión:

```bash
sudo apt update
sudo apt install openssl
openssl version
```

## Comando general de cifrado

```bash
openssl enc -<algoritmo> -<modo> -salt -in <entrada> -out <salida>
```

- `openssl enc`: indica operación de cifrado/descifrado.
- `-<algoritmo>`: algoritmo de cifrado (p. ej. `aes-256-cbc`).
- `-<modo>`: `-e` para cifrar, `-d` para descifrar.
- `-salt`: añade aleatoriedad y dificulta ataques de diccionario.
- `-in` / `-out`: archivo de entrada y de salida.

## Ejercicio 1 — Cifrar/descifrar un archivo de texto

```bash
# Crear archivo
echo "Este es un archivo de texto confidencial." > texto.txt

# Cifrar (pedirá contraseña)
openssl enc -aes-256-cbc -salt -in texto.txt -out texto_encriptado.enc

# Descifrar (misma contraseña)
openssl enc -d -aes-256-cbc -in texto_encriptado.enc -out texto_desencriptado.txt

# Verificar
cat texto_desencriptado.txt
```

## Ejercicio 2 — Cifrar/descifrar un archivo binario

```bash
cp /usr/share/pixmaps/gnome-logo.png imagen.png
openssl enc -aes-256-cbc -salt -in imagen.png -out imagen_encriptada.png.enc
openssl enc -d -aes-256-cbc -in imagen_encriptada.png.enc -out imagen_desencriptada.png
```

Abre la imagen descifrada con un visor para comprobar que se restauró bien.

## Ejercicio 3 — Cifrar/descifrar una carpeta

```bash
# Crear y poblar la carpeta
mkdir carpeta_original
echo "Archivo 1" > carpeta_original/archivo1.txt
echo "Archivo 2" > carpeta_original/archivo2.txt

# Empaquetar y comprimir
tar -cvf carpeta_original.tar carpeta_original/
gzip carpeta_original.tar

# Cifrar el comprimido
openssl enc -aes-256-cbc -salt -in carpeta_original.tar.gz -out carpeta_encriptada.tar.gz.enc

# Descifrar
openssl enc -d -aes-256-cbc -in carpeta_encriptada.tar.gz.enc -out carpeta_encriptada.tar.gz

# Descomprimir y extraer
gunzip carpeta_encriptada.tar.gz
tar -xvf carpeta_encriptada.tar
ls carpeta_original/
```

## Algoritmos de cifrado simétrico

Una **única clave** cifra y descifra; emisor y receptor la comparten. Es rápido, pero depende de proteger la clave secreta.

| Algoritmo | Notas |
|---|---|
| AES | El más usado hoy. Claves de 128/192/256 bits. Modos CBC, GCM, etc. |
| DES | Estándar antiguo, inseguro por su clave de 56 bits. 3DES aplica DES tres veces. |
| Blowfish | Bloque de 64 bits, clave variable (hasta 448 bits). Rápido. |
| Twofish | Sucesor de Blowfish, bloques de 128 bits, claves hasta 256 bits. |

## Algoritmos de cifrado asimétrico

Dos claves: **pública** para cifrar y **privada** para descifrar. Más lento que el simétrico, pero seguro para transmitir claves.

| Algoritmo | Notas |
|---|---|
| RSA | Cifrado y firmas digitales. Seguridad basada en la factorización de números primos grandes. |
| ECC | Curvas elípticas; seguridad similar a RSA con claves mucho más cortas. Eficiente en dispositivos limitados. |
| ElGamal | Basado en el problema del logaritmo discreto. Cifrado de clave pública y firmas. |

## Algoritmos de hashing

No cifran, sino que crean una representación fija (hash) para verificar integridad y almacenar contraseñas.

| Algoritmo | Notas |
|---|---|
| MD5 | Hash de 128 bits. Inseguro para usos críticos. |
| SHA-1 | Hash de 160 bits. También considerado inseguro. |
| SHA-2 | Variantes SHA-224/256/384/512. Seguro y muy usado. |
| SHA-3 | Último estándar, mejor resistencia a ataques que SHA-2. |

## Modos de operación

| Modo | Descripción |
|---|---|
| ECB | Cifra bloques de forma independiente. No recomendado para datos sensibles. |
| CBC | Encadena cada bloque con el anterior cifrado; más seguro que ECB. |
| CFB | Opera sobre bloques de tamaño variable; útil para flujos. |
| OFB | Genera un flujo de cifrado para aplicar a los datos. |
| GCM | Cifrado + autenticación; ideal cuando se requieren ambos. |

> [!warning] Algoritmos obsoletos
> Evita DES, MD5 y SHA-1 en sistemas nuevos. Prefiere AES (GCM) para cifrado simétrico y SHA-2/SHA-3 para hashing.

---
📎 Guías: ![[OpenSSL.pdf]]

🔗 Relacionado: [[GPG - cifrado asimétrico y firmas]]
