---
tags:
  - seguridad-web
  - owasp
  - type-juggling
aliases:
  - Type Juggling (PHP)
  - Type Juggling
  - Malabarismo de tipos
---

# Type Juggling (PHP)

**El type juggling es un fallo derivado de la conversión automática de tipos en lenguajes débilmente tipados como PHP: al usar comparación laxa (`==`) en vez de estricta (`===`), el atacante controla un valor para que la comparación devuelva un resultado inesperado.** Permite saltar comprobaciones de autenticación, HMAC o contraseñas.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
La comparación laxa (`==`/`!=`) solo compara "el mismo valor" tras convertir tipos; la estricta (`===`/`!==`) compara valor y tipo. PHP convierte cadenas a números cuando lo cree necesario, generando equivalencias falsas. Caso clave: un hash que empieza por `0e` seguido solo de dígitos se interpreta como notación científica (`0 * 10^n = 0`), por lo que dos hashes así son "iguales" entre sí y a `0`.

> En PHP 8 la RFC "Saner string to number comparisons" ya no convierte cadenas no numéricas a número, eliminando muchas de estas colisiones.

## Detección y pruebas
Busca comparaciones de tokens, contraseñas o HMAC con `==`/`!=` donde controles uno de los operandos. Tabla de equivalencias laxas (PHP < 8):

| Comparación | Resultado |
|---|---|
| `'123a' == 123` | true |
| `'abc' == 0` | true (PHP < 8) |
| `'0e123' == '0e456'` | true (magic hashes) |
| `'' == 0 == false == NULL` | true |
| `0 == false` | true |

`md5([])` y `sha1([])` devuelven `NULL`, lo que puede saltar comprobaciones si se pasa un array.

## Explotación / payloads
**Magic hashes** (cadenas cuyo hash es de la forma `0e[dígitos]`, comparan como iguales):

```php
var_dump(md5('240610708') == md5('QNKCDZO'));   // bool(true)
var_dump(sha1('aaroZmOk') == sha1('aaK1STfY'));  // bool(true)
```

**Bypass de HMAC en cookie** con comparación laxa: el atacante fija `hmac = "0"` y fuerza por fuerza bruta un `expiration` cuyo `hash_hmac('md5', ...)` empiece por `0e` y solo dígitos:

```php
for ($i = 1424869663; $i < 1835970773; $i++) {
    $out = hash_hmac('md5', 'admin|'.$i, '');
    if (str_starts_with($out, '0e') && $out == 0) { echo "$i - $out"; break; }
}
// cookie: username=admin, expiration=1539805986, hmac=0
```

También afecta a bypass tipo `0 == strcmp($_GET['user'], $pass)` cuando `strcmp` devuelve `NULL` (PHP < 8).

## Mitigación
- Usar **siempre comparación estricta** (`===`/`!==`) para tokens, contraseñas y firmas.
- Comparar hashes y HMAC con funciones de tiempo constante: `hash_equals()`, `password_verify()`.
- Validar y forzar el tipo de la entrada (no aceptar arrays donde se espera string).
- Actualizar a PHP 8+, que corrige gran parte de estas conversiones.

## Herramientas
- **Loose-Compare-Tables** (Hakumarachi) — tablas de comparación laxa por lenguaje (PHP, MySQL, Node, Python...).
- Listas de **magic hashes** de @spaze para MD4/MD5/SHA-1/SHA-256.

---
🔗 Relacionado: [[Aleatoriedad insegura]] · [[Errores de lógica de negocio]] · [[Deserialización insegura]]
📚 Fuente: [PayloadsAllTheThings — Type Juggling](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling) (MIT, © Swissky)
