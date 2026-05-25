---
tags:
  - criptografia
  - gpg
  - openpgp
aliases:
  - GnuPG
  - GPG
---

# GPG: cifrado asimétrico y firmas

**GPG (GNU Privacy Guard)** — herramienta de cifrado que asegura la **confidencialidad, integridad y autenticidad** de los datos mediante **criptografía asimétrica**. Usa dos claves: una **pública**, que se comparte libremente, y una **privada**, que se mantiene en secreto. Es compatible con el estándar **OpenPGP**.

## ¿Para qué se usa?

| Función | Descripción |
|---|---|
| Cifrado de mensajes y archivos | Solo quien posee la clave privada correspondiente puede descifrar el contenido. |
| Firmas digitales | Verifican la autenticidad e integridad de un mensaje o archivo. |
| Verificación de firmas | Confirma que un contenido fue firmado por una clave específica y no se alteró. |
| Gestión de claves | Generar, exportar, importar y revocar claves públicas y privadas. |
| Autenticación de identidad | Garantiza que los cambios (p. ej. en código fuente) provienen de fuentes confiables. |

## Cómo funciona la criptografía asimétrica

- La **clave pública** cifra y verifica firmas; se distribuye sin riesgo.
- La **clave privada** descifra y firma; nunca se comparte y se protege con contraseña.
- Para enviarte algo cifrado, otra persona usa **tu clave pública**; solo tú lo descifras con tu **clave privada**.

## Generar y verificar claves

Crea tu par de claves (pública/privada):

```bash
gpg --gen-key
```

Durante el proceso se eligen tipo, tamaño y fecha de expiración de la clave, se asocia nombre y correo, y se define una contraseña para proteger la clave privada.

Lista las claves del llavero para verificar que se generaron bien:

```bash
gpg -k
```

## Intercambio de claves públicas

Exporta tu clave pública en formato ASCII para compartirla:

```bash
gpg -a --export -o <archivo_salida> <identificador>
```

Importa la clave pública de otra persona a tu llavero (necesaria para cifrarle mensajes):

```bash
gpg --import <archivo_clave_publica>
```

> [!tip] Identificador
> El `<identificador>` puede ser el UID, el nombre o el correo asociado a la clave.

## Cifrar y descifrar

Cifra un archivo para que solo el destinatario lo abra con su clave privada:

```bash
gpg --encrypt --recipient <uid_o_email> <archivo_texto_claro>
```

Descifra un archivo recibido usando tu clave privada:

```bash
gpg --output <salida_texto_claro> --decrypt <archivo_cifrado>
```

## Firmar y verificar firmas

Firma un documento para acreditar tu autoría e integridad:

```bash
gpg --sign <archivo>
```

Verifica la firma de un documento recibido:

```bash
gpg --verify <archivo_firmado>
```

La verificación confirma que el archivo fue firmado por la clave privada correspondiente y que no se modificó desde entonces.

## Revocar una clave

Si la clave privada se ve comprometida o ya no la necesitas, genera un certificado de revocación que la deshabilita:

```bash
gpg --gen-revoke <uid>
```

> [!warning] Protege tu clave privada
> Una clave privada comprometida permite suplantar tu identidad y descifrar tus mensajes. Guárdala cifrada con una contraseña fuerte y prepara con antelación tu certificado de revocación.

---
📎 Guías: ![[GPG-Cifrado-Asimetrico-Firmas.pdf]]

🔗 Relacionado: [[OpenSSL]]
