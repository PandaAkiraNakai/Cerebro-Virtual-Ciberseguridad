---
tags:
  - seguridad-web
  - owasp
  - business-logic
aliases:
  - Errores de lógica de negocio
  - Business Logic Errors
  - Fallos de lógica de negocio
---

# Errores de lógica de negocio

**Los errores de lógica de negocio son fallos en el diseño del flujo de la aplicación —no en la implementación técnica— que permiten usar funciones legítimas de forma no prevista para obtener una ventaja: precios negativos, saltarse pasos, abusar de descuentos o eludir comprobaciones de proceso.**

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
A diferencia de SQLi o XSS, aquí no hay un payload "malicioso": las peticiones son técnicamente válidas, pero rompen suposiciones que el desarrollador dio por sentadas (que un precio nunca será negativo, que el usuario seguirá los pasos en orden, que el cliente no manipulará campos ocultos). Suelen aparecer en checkout, cupones, transferencias, registros multi-paso y límites de uso. Son difíciles de detectar con escáneres porque dependen del **contexto del negocio**.

## Detección y pruebas
Modela el flujo esperado y pregúntate "¿qué pasa si...?": valores negativos o enormes, omitir un paso, repetir una acción, cambiar moneda/cantidad/estado, llegar a un endpoint fuera de orden. Manipula campos que el cliente "no debería" tocar (precio, rol, total) y observa si el servidor los acepta. Combina con [[IDOR]] y [[Race Conditions]] para amplificar.

## Explotación / payloads
**Valores fuera de rango / negativos:**

```
POST /comprar    cantidad=-5&precio=100      # total negativo -> abono al atacante
POST /transferir monto=-1000                  # invertir el sentido del cargo
```

**Confianza en campos del cliente:**

```
POST /checkout   total=0.01                    # precio enviado por el cliente
POST /cupon      codigo=DESC50&codigo=DESC50    # apilar descuentos
```

**Salto de pasos del flujo:**

```
# Ir directo al paso "pago confirmado" sin pasar por "pago"
POST /pedido/confirmar?id=999&estado=pagado
```

**Abuso de límites:** redondeos, conversiones de divisa, reembolsos repetidos, registro de cuentas con privilegios por defecto.

## Mitigación
- Validar **toda** la lógica en el servidor; nunca confiar en precios, totales, roles o estados enviados por el cliente.
- Imponer máquinas de estado: rechazar transiciones fuera de orden y verificar precondiciones de cada paso.
- Definir y comprobar invariantes de negocio (rangos, no-negatividad, idempotencia) y registrar anomalías.
- Hacer threat modeling centrado en el flujo y pruebas de aceptación con casos abusivos.

## Herramientas
- **Burp Suite** (Repeater) — manipulación manual del flujo; es trabajo mayoritariamente manual.
- Diagramas de flujo / threat modeling para enumerar suposiciones.

---
🔗 Relacionado: [[Race Conditions]] · [[Type Juggling (PHP)]] · [[IDOR]] · [[Mass Assignment]]
📚 Fuente: [PayloadsAllTheThings — Business Logic Errors](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Business%20Logic%20Errors) (MIT, © Swissky)
