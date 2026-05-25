---
tags:
  - seguridad-web
  - owasp
  - hpp
aliases:
  - HTTP Parameter Pollution (HPP)
  - HTTP Parameter Pollution
  - HPP
  - Contaminación de parámetros HTTP
---

# HTTP Parameter Pollution (HPP)

**El HTTP Parameter Pollution consiste en enviar varias veces el mismo parámetro en una petición para explotar las diferencias en cómo cada componente (servidor, framework, WAF, backend) decide cuál valor usar.** Esas discrepancias permiten saltar validaciones, envenenar lógica o burlar filtros de seguridad.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
No existe un estándar único sobre qué hacer con parámetros duplicados, así que cada tecnología elige distinto: algunas toman el **primero**, otras el **último**, y otras los **concatenan**. Si el WAF valida el primer valor pero el backend usa el último, el atacante "esconde" el payload en la copia que el WAF ignora. Hay dos variantes: **server-side** (la lógica del servidor) y **client-side** (inyección en enlaces/formularios que se reflejan).

## Detección y pruebas
Duplica un parámetro con valores distintos y observa cuál procesa la aplicación. Mapea la cadena de componentes (proxy → WAF → app → DB) y prueba si difieren. Es especialmente útil cuando hay un control de seguridad por delante: lo que el filtro no ve, puede llegar limpio al backend.

## Explotación / payloads
**Comportamiento por tecnología** (referencia):

```
ASP.NET / IIS        -> concatena con coma:   par=val1,val2
PHP / Apache         -> último valor:         par=val2
JSP / Tomcat         -> primer valor:         par=val1
Python / Flask       -> primer valor
Node.js / Express    -> array:               par=[val1,val2]
```

**Bypass de WAF / validación** (el filtro lee uno, el backend otro):

```
GET /buscar?q=inofensivo&q=' OR '1'='1
POST  user=admin&user=victima
```

**Manipulación de lógica de negocio:**

```
/transferir?monto=10&monto=10000      # un control valida 10, otro ejecuta 10000
/api?role=user&role=admin
```

**HPP del lado cliente** — inyectar en parámetros reflejados en enlaces:

```
?id=123%26admin=true     # el %26 se decodifica y añade un parámetro extra
```

## Mitigación
- Normalizar parámetros duplicados de forma explícita y coherente en toda la pila; rechazar peticiones con parámetros repetidos cuando no se esperen.
- Validar y saner cada valor en el mismo punto que lo consume el backend (no solo en el borde).
- Usar APIs que devuelvan listas para parámetros repetidos y tratarlas conscientemente.
- Homogeneizar el parseo entre WAF/proxy y aplicación para eliminar discrepancias.

## Herramientas
- **Burp Suite** (Repeater) — duplicar parámetros y comparar respuestas.
- **Param Miner** (extensión de Burp) — descubrir parámetros y comportamientos ocultos.

---
🔗 Relacionado: [[Inyección CRLF]] · [[Parámetros ocultos]] · [[IDOR]] · [[Inyección SQL (SQLi)]]
📚 Fuente: [PayloadsAllTheThings — HTTP Parameter Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/HTTP%20Parameter%20Pollution) (MIT, © Swissky)
