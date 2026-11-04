---
tags:
  - meta
  - fuentes
  - licencias
aliases:
  - Fuentes
  - Licencias
  - Atribución
---

# 📜 Fuentes y Licencias

Este vault reúne y reorganiza material de estudio de **redes, sistemas y ciberseguridad**. Cada nota es una reescritura/curaduría propia; abajo se acreditan las fuentes y sus licencias. Punto de entrada del vault: [[🗺️ MOC Ciberseguridad y Redes]].

> [!warning] Uso ético y autorizado
> El contenido ofensivo (pentesting, payloads, post-explotación) es **educativo** y está pensado para pruebas en sistemas **propios o con autorización explícita por escrito**. Usar estas técnicas contra sistemas de terceros sin permiso es **ilegal**. Quien usa el vault es responsable de cumplir la ley aplicable.

## Fuentes

### PayloadsAllTheThings
- Repo: <https://github.com/swisskyrepo/PayloadsAllTheThings>
- Licencia: **MIT** — © 2019 Swissky
- Uso en el vault: base de las secciones de **seguridad web** (25) y la metodología ofensiva posterior (recon, post-explotación, Active Directory, cracking, cloud). Las notas son adaptaciones condensadas; los payloads se incluyen como referencia.

### pulentoski/Herramientas-de-pentesting
- Repo: <https://github.com/pulentoski/Herramientas-de-pentesting>
- Uso en el vault: base de las secciones **27 · Cracking de Contraseñas** (Hashcat, John the Ripper, Hydra/Medusa, extracción de hashes `*2john`) y **28 · Esteganografía** (StegSeek/Steghide), más las fichas de **WPScan** y **CrackMapExec/NetExec**. Las notas son adaptaciones condensadas y reescritas al formato del vault; no se importan los exploits ejecutables del repo original.

### aw-junaid/Kali-Linux *(pendiente — fase posterior)*
- Repo: <https://github.com/aw-junaid/Kali-Linux> — Licencia **MIT**
- Uso previsto: catálogo de **herramientas de Kali** (sección 31).

### Metasploit Framework *(pendiente — fase posterior)*
- Repo: <https://github.com/rapid7/metasploit-framework> — Licencia tipo BSD (Metasploit Framework License)
- Uso previsto: solo **notas de uso** (msfconsole, msfvenom, módulos). No se importa código ni módulos.

### pulentoski/repositorio-configuraciones (secciones 00–23 y 29)
- Repo: <https://github.com/pulentoski/repositorio-configuraciones>
- Licencia: el repositorio no declara una licencia general.
- Uso en el vault: base de redes, Cisco, Linux, IoT, cloud y servicios (00–23) y de la sección **29 · Auditoría y Gestión de Riesgos**. La actualización de noviembre de 2026 incorporó NAT, FHRP, VTP/MVRP, STP/RSTP/MSTP, OSPF con autenticación, Telnet, FTDv en CML, pfSense en VirtualBox, TACACS+ con tac_plus-ng, Zabbix 7.4, FreePBX 16, iperf3, red por distribución en Linux, hardening de SSH con MFA, RadioMobile y la guía de Suricata.
- Las notas son reescrituras condensadas: se quitaron datos personales e institucionales de los originales, se corrigieron errores técnicos y no se copian binarios, imágenes de equipos ni PDF.
- **Excepciones:** el archivo de SNMP v2/v3 declara licencia CC BY-NC-ND 4.0 (sin obras derivadas), así que no se adaptó; la nota [[SNMPv3 en Cisco]] se escribió desde la documentación general de Cisco. La guía de Zabbix en un solo contenedor no se integró porque la imagen que usa no incluye la base de datos ni la interfaz web.
- Las guías originales en PDF se conservan **solo localmente** y **no se publican**.

## Sobre este vault
Las adaptaciones propias se comparten con **fines educativos**. El material de terceros conserva su licencia original (ver arriba); se mantiene la atribución y el aviso de copyright correspondiente en cada nota derivada.
