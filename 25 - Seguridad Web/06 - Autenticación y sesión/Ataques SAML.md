---
tags:
  - seguridad-web
  - owasp
  - saml
aliases:
  - SAML
  - SAML Injection
  - Ataques SAML
  - XML Signature Wrapping
---

# Ataques SAML

**SAML es un estándar para intercambiar datos de autenticación y autorización entre un proveedor de identidad (IdP) y uno de servicio (SP), muy usado en SSO; una implementación deficiente permite falsificar aserciones y suplantar usuarios.** Una respuesta SAML contiene `<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">`.

> [!warning] Uso autorizado
> Técnicas para pruebas en sistemas propios o con autorización explícita. Ver [[📜 Fuentes y Licencias]].

## ¿Cómo funciona?
Tras autenticar al usuario, el IdP emite una **aserción SAML** (XML) firmada digitalmente que indica quién es el usuario (`NameID`) y sus atributos. El SP confía en esa aserción si la firma es válida. La mayoría de ataques explotan que el SP no verifica correctamente esa firma, que se procesan varias aserciones, o que el parser XML altera el contenido tras la verificación.

## Detección y pruebas
- Intercepta la respuesta SAML (suele ir en Base64 dentro de un parámetro POST) y decodifícala.
- Identifica `Issuer`, `Assertion`, `NameID` y el bloque `<ds:Signature>`.
- Prueba a quitar la firma, a clonar aserciones (XSW), a insertar comentarios en `NameID` y a añadir entidades XML.
- Herramientas como **SAMLRaider** automatizan la edición y los ataques XSW.

## Explotación / payloads
**Firma no validada / firma autofirmada:** si la firma no la emite una CA real, puede clonarse o sustituirse por un certificado autofirmado propio.

**Firma omitida (signature stripping):** muchas configuraciones, si falta el bloque de firma, **no realizan verificación alguna**. Se forja una aserción bien formada sin firmar con `NameID=admin`:

```xml
<saml2:Subject>
  <saml2:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified">admin</saml2:NameID>
</saml2:Subject>
```
"Aceptar aserciones SAML sin firmar es aceptar un usuario sin comprobar la contraseña."

**XML Signature Wrapping (XSW):** algunos SP validan una firma legítima pero usan otra aserción no firmada (no controlan múltiples aserciones ni el orden). Se añade una **aserción forjada (FA)** junto a la legítima firmada (LA):

```xml
<SAMLResponse>
  <FA ID="evil"><Subject>Atacante</Subject></FA>
  <LA ID="legitimate">
    <Subject>Usuario Legitimo</Subject>
    <LAS><Reference URI="legitimate"/></LAS>
  </LA>
</SAMLResponse>
```
Variantes **XSW1–XSW8** según se clone la Response o la Assertion y dónde se ubique la copia (antes/después de la firma, en Extensions, en Object). Fue el caso de GitHub Enterprise.

**Comentarios en NameID (CVE-2017-11427 y familia):** un comentario XML dentro del `NameID` hace que distintas librerías lean solo una parte del valor, permitiendo suplantar a otro usuario. Afectó a python-saml, ruby-saml, saml2-js, OmniAuth-SAML, Shibboleth, Duo.

```xml
<NameID>user@user.com<!--XMLCOMMENT-->.evil.com</NameID>
```
El SP procesa solo `user@user.com`.

**XXE en la aserción:** entidades XML que se resuelven tras la verificación de firma (la firma no cambia, pero el valor sí al parsear):

```xml
<!DOCTYPE Response [ <!ENTITY s "s"> <!ENTITY f1 "f1"> ]>
...
<saml2:AttributeValue>&s;taf&f1;</saml2:AttributeValue>
```

**XSLT en `Transform`:** un stylesheet malicioso dentro de `<ds:Transforms>` puede leer ficheros (`/etc/passwd`) y exfiltrarlos a un servidor del atacante.

## Mitigación
- Verificar **siempre** la firma de la aserción y **rechazar respuestas sin firmar**.
- Validar que existe una única aserción firmada y comprobar que el `Reference URI` de la firma apunta exactamente a la aserción usada (mitiga XSW).
- Usar librerías SAML actualizadas que extraigan el texto completo del nodo (corrección del bug de comentarios).
- Deshabilitar entidades externas y DTD en el parser XML (anti-XXE) y deshabilitar transformaciones XSLT.
- Confiar solo en certificados de IdP en lista blanca, no autofirmados arbitrarios; validar `Audience`, `NotBefore`/`NotOnOrAfter` y `Destination`.

## Herramientas
- **SAMLRaider** (extensión de Burp) — edición SAML y ataques XSW.
- **XSW** (d0ge, Burp) — automatiza XML Signature Wrapping.
- **ZAP SAML Support** — detección, edición y fuzzing de peticiones SAML.

---
🔗 Relacionado: [[OAuth mal configurado]] · [[Account Takeover]] · [[XXE (XML External Entity)]] · [[JWT (JSON Web Token)]]
📚 Fuente: [PayloadsAllTheThings — SAML Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SAML%20Injection) (MIT, © Swissky)
