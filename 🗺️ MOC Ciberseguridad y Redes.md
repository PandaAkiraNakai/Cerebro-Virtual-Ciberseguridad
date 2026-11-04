---
tags:
  - moc
  - redes
  - ciberseguridad
aliases:
  - Índice
  - MOC
  - Inicio
---

# 🗺️ MOC — Ciberseguridad y Redes

> [!info] Origen y fuentes
> Reorganizado y reescrito a partir de repositorios públicos. Atribución, licencias y aviso de uso ético en [[📜 Fuentes y Licencias]]. Las guías originales en PDF se conservan localmente y **no se publican** (privacidad); cada nota es autocontenida.

---

## ⚡ Navegación rápida

| Área | Saltar a |
|------|----------|
| 🌐 Redes y Cisco | [Fundamentos](#00--fundamentos) · [Cisco IOS](#01--cisco-ios--operación) · [Direccionamiento](#02--direccionamiento-y-dhcp) · [Enrutamiento](#03--enrutamiento-dinámico) · [Switching](#04--switching-y-vlan) · [ACL](#05--listas-de-control-de-acceso-acl) · [QoS](#06--calidad-de-servicio-qos) |
| 🔒 Seguridad Cisco | [Seguridad Cisco](#07--seguridad-cisco) · [Conexiones remotas](#08--conexiones-remotas) · [MikroTik](#09--mikrotik) |
| 💻 Sistemas | [Linux](#10--sistemas-operativos-linux) · [Contenedores y AD](#11--contenedores-y-directorio-activo) |
| 🛡️ Defensiva | [Criptografía](#12--criptografía) · [Firewalls](#13--firewalls) · [IDS/IPS](#14--ids-e-ips) |
| 📋 Gobierno | [Auditoría y riesgos](#29--auditoría-y-gestión-de-riesgos) |
| ⚔️ Ofensiva | [Reconocimiento](#24--reconocimiento) · [Pentesting](#15--pentesting) · [Cracking](#27--cracking-de-contraseñas) · [Esteganografía](#28--esteganografía) · [Post-Explotación](#26--post-explotación) · [Phishing](#16--phishing) · [Seguridad Web](#25--seguridad-web-owasp--payloadsallthethings) |
| 📊 Monitoreo | [Monitoreo](#17--monitoreo-y-rendimiento) |
| 📡 IoT / Cloud | [IoT](#18--iot) · [Meshtastic](#19--meshtastic) · [Cloud](#20--cloud) · [Servicios](#21--servicios) |
| ⚙️ Dev | [Automatización](#22--automatización) · [Programación](#23--programación) |

---

## 🔍 Índice por herramienta

| Herramienta | Nota |
|-------------|------|
| Ansible | [[Ansible]] |
| ARP Spoofing / Bettercap | [[ARP Spoofing y MitM]] |
| AWS | [[AWS - VPC, subredes, balanceo y autoescalado]] |
| Docker | [[Comandos Docker]] · [[ThingsBoard en Docker]] · [[Laboratorio DVWA (CTF)]] · [[GNS3 - laboratorios de red virtualizados]] |
| ESP8266 | [[ESP8266 sketches y firmware]] |
| fail2ban / UFW / Lynis / MFA TOTP en SSH | [[08 - Seguridad del servidor]] |
| FreePBX / Asterisk / Linphone | [[FreePBX 16 en VirtualBox]] · [[PBX (Asterisk - FreePBX) en AWS]] · [[QoS en router y switch con FreePBX]] |
| FTDv / FDM (Cisco Secure Firewall) | [[FTDv en Cisco CML]] |
| GNS3 / vIOS-L2 | [[GNS3 - laboratorios de red virtualizados]] |
| Gobuster / WFuzz | [[Fuzzing de Directorios y Subdominios]] |
| GoPhish | [[GoPhish]] |
| GPG | [[GPG - cifrado asimétrico y firmas]] |
| Hydra | [[Hydra y Medusa]] · [[Pentesting en Redes]] |
| iperf3 | [[iperf3 test de velocidad real]] |
| netplan / nmcli / ifupdown | [[Configuración de red por distribución]] |
| iptables | [[iptables]] |
| Kali Linux | [[Laboratorio Kali Linux en Windows]] |
| Metasploit / msfvenom | [[Metasploit y Payload Reverse TCP]] · [[Metasploit — Módulos avanzados]] |
| Meshtastic / LoRa | [[Meshtastic comandos y scripts]] |
| MQTT / Mosquitto | [[MQTT con Mosquitto en Debian]] · [[Panel y comandos MQTT (Python)]] · [[Servidor MQTT a MySQL (Python)]] · [[Envío de mensajes MQTT desde PHP]] |
| MySQL / phpMyAdmin | [[MySQL + phpMyAdmin]] · [[Servidor MQTT a MySQL (Python)]] |
| Nmap | [[Nmap]] · [[Pentesting en Redes]] · [[Reconocimiento Activo de Red]] |
| nuclei | [[Reconocimiento Web]] |
| subfinder / amass | [[Reconocimiento Web]] · [[Reconocimiento Pasivo (OSINT)]] |
| theHarvester | [[Reconocimiento Pasivo (OSINT)]] |
| Shodan / FOFA | [[Reconocimiento Pasivo (OSINT)]] |
| LinPEAS / WinPEAS | [[Escalada de Privilegios Linux]] · [[Escalada de Privilegios Windows]] |
| GTFOBins | [[Escalada de Privilegios Linux]] |
| Chisel / Ligolo-ng | [[Pivoting de Red]] |
| proxychains | [[Pivoting de Red]] |
| Hashcat | [[Hashcat]] |
| John the Ripper / *2john | [[John the Ripper]] · [[Proceso de Extracción de Hashes]] |
| Medusa | [[Hydra y Medusa]] |
| CrackMapExec / NetExec | [[CrackMapExec - NetExec]] |
| WPScan | [[WPScan]] |
| sqlmap | [[sqlmap]] |
| StegSeek / Steghide / zsteg | [[Esteganografía y análisis de archivos]] |
| binwalk / exiftool | [[Esteganografía y análisis de archivos]] |
| Impacket | [[Escalada de Privilegios Windows]] · [[Active Directory — Ataques]] |
| OpenSSL | [[OpenSSL]] |
| pfSense | [[pfSense en VirtualBox]] · [[Portal cautivo en pfSense]] |
| RadioMobile | [[Radioenlaces y simulación en RadioMobile]] |
| Raspberry Pi | [[SSH en Raspberry Pi OS]] · [[Raspberry Pi pantalla OLED]] |
| Snort | [[Snort]] |
| Splunk | [[Splunk Enterprise]] |
| SSH | [[SSH en Cisco]] · [[SSH en Linux]] · [[SSH en Raspberry Pi OS]] |
| Suricata / suricata-update | [[Suricata]] |
| tac_plus-ng (TACACS+) | [[TACACS+ con tac_plus-ng]] |
| ThingsBoard | [[ThingsBoard en Docker]] · [[ESP8266 sketches y firmware]] |
| TVHeadend | [[Servidor IPTV con TVHeadend]] |
| Wireshark | [[Wireshark]] |
| Zabbix / SNMP | [[Zabbix 7.4 con MySQL]] · [[SNMPv3 en Cisco]] · [[Zabbix y SNMP en Cisco]] · [[Zabbix y SNMP en MikroTik]] |

---

# 🌐 Redes

## 00 · Fundamentos
- [[Capacity Planning y QoS]] — dimensionamiento de ancho de banda, overhead, codecs y métricas de QoS.
- [[GNS3 - laboratorios de red virtualizados]] — emulación de topologías con IOS real, VM y contenedores; plantilla vIOS-L2, consolas, persistencia y API.
- [[Radioenlaces y simulación en RadioMobile]] — presupuesto de enlace, zona de Fresnel y parámetros de simulación.

## 01 · Cisco IOS — Operación
- [[Configuración básica de equipos Cisco]] — hostname, contraseñas, líneas, cifrado.
- [[Protocolos de descubrimiento y sincronización (CDP, LLDP, NTP)]]
- [[Carga de IOS por ROMMON y TFTP]] — recuperación de imagen `.bin`.
- [[Recuperación de contraseña por ROMMON]] — registro `0x2142`.
- [[Recuperación de equipos Cisco (referencia rápida)]]

## 02 · Direccionamiento y DHCP
- [[Direcciones IP estáticas y rutas estáticas]]
- [[Servicio DHCP en Cisco]]
- [[NAT y PAT en Cisco]] — NAT estática, dinámica y PAT (overload) con inside/outside.

## 03 · Enrutamiento dinámico
- [[OSPF]] — link-state, costo, wildcard, autenticación MD5/HMAC-SHA y diagnóstico de vecindades.
- [[EIGRP]] — híbrido, DUAL, AS.
- [[Redistribución EIGRP ↔ OSPF]]

## 04 · Switching y VLAN
- [[VLAN y enrutamiento Inter-VLAN]] — tipos y rangos de VLAN, troncal 802.1Q, SVI y router-on-a-stick.
- [[Gestión de VLAN con VTP y MVRP]] — modos VTP, revision number y borrado masivo, GVRP/MVRP.
- [[Spanning Tree Protocol (STP)]] — STP, RSTP, Rapid PVST+ y MSTP; roles, estados, PortFast y BPDU Guard.
- [[EtherChannel con LACP]] — LACP, PAgP y modo On; balanceo por hash y Port-Channel de capa 3.
- [[FHRP - HSRP, VRRP y GLBP]] — gateway virtual redundante, preempt, tracking e IP SLA.

## 05 · Listas de Control de Acceso (ACL)
- [[ACL - Conceptos]] — tipos, ubicación, dirección.
- [[ACL estándar]]
- [[ACL extendida]]

## 06 · Calidad de Servicio (QoS)
- [[QoS para voz sin switch administrable]]
- [[QoS en router y switch con FreePBX]]

## 07 · Seguridad Cisco
- [[Seguridad en la Capa 2]] — mapa de ataques y mitigaciones L2.
- [[Port Security]]
- [[DHCP Snooping]]
- [[AAA en Cisco]]
- [[Usuarios y niveles de privilegio]]
- [[SSH en Cisco]] — SSHv2, claves RSA de 2048 bits, VTY y errores comunes.
- [[TACACS+ con tac_plus-ng]] — AAA centralizado en Ubuntu/Debian: perfiles, autorización por comando y accounting.
- [[Acceso y autenticación segura en Cisco]]

## 08 · Conexiones remotas
- [[SSH en Linux]]
- [[SSH en Raspberry Pi OS]]
- [[Telnet en Cisco IOS]] — líneas VTY, `login` frente a `login local` y migración a SSH.

## 09 · MikroTik
- [[PPPoE para FTTH (MikroTik)]] — crear un ISP / perfiles de navegación.

---

# 💻 Sistemas y administración

## 10 · Sistemas Operativos Linux
- [[01 - Inspección del sistema]] — kernel, hardware, CPU/RAM, dispositivos y módulos.
- [[02 - Red y conectividad]] — ip, ss, nmap, firewall, DNS y diagnóstico de red.
- [[03 - Procesos y rendimiento]] — top/htop, ps, señales, load average y OOM killer.
- [[04 - Almacenamiento y filesystems]] — df, particionado, LVM, SMART y fstab.
- [[05 - Usuarios, grupos y permisos]] — useradd, chmod, sudoers por rol, escape a shell, ACL y permisos especiales.
- [[06 - Servicios con systemd]] — systemctl, journalctl, targets, cron y timers.
- [[07 - Logs y auditoría]] — journalctl, grep/awk/sed, auditd y logrotate.
- [[08 - Seguridad del servidor]] — llaves SSH, MFA TOTP, AllowGroups, fail2ban, UFW, Lynis y SSL/TLS.
- [[09 - Backups y recuperación]] — rsync, tar, dd, Borg, Restic y snapshots LVM.
- [[10 - Scripting y automatización]] — Bash: variables, bucles, funciones y manejo de errores.
- [[11 - Virtualización y contenedores]] — Proxmox (qm/pct), Docker, LXC y virsh.
- [[12 - Troubleshooting y diagnóstico]] — strace, tcpdump, lsof, perf e incidentes.
- [[Configuración de red por distribución]] — netplan, ifupdown, NetworkManager y TinyCore; IP estática, rutas y bridges.

## 11 · Contenedores y Directorio Activo
- [[Comandos Docker]] — imágenes, contenedores, volúmenes, redes y Compose.
- [[Unir Debian 12 a Active Directory]] — integración con AD vía SSSD, Kerberos, PAM y Samba.

---

# 🛡️ Ciberseguridad defensiva

## 12 · Criptografía
- [[GPG - cifrado asimétrico y firmas]] — cifrado/descifrado, firmas digitales e intercambio de claves con GnuPG (OpenPGP).
- [[OpenSSL]] — cifrado de archivos, certificados SSL/TLS, hashing y referencia de algoritmos.

## 13 · Firewalls
- [[Portal cautivo en pfSense]] — autenticación de usuarios, certificado, DNS Resolver y Captive Portal.
- [[iptables]] — tablas, cadenas y reglas de filtrado en Linux; políticas por defecto y mitigación.
- [[pfSense en VirtualBox]] — instalación de pfSense CE con WAN/LAN, ZFS y DHCP para laboratorio.
- [[FTDv en Cisco CML]] — Cisco Secure Firewall virtual con FDM: cableado, Easy Setup y políticas.

## 14 · IDS e IPS
- [[Snort]] — IDS/IPS: modos, reglas, preprocesadores e instalación en modo IPS.
- [[Suricata]] — IDS/IPS: `suricata.yaml`, reglas propias, `suricata-update`, modo IPS con NFQUEUE, `eve.json` con jq y rendimiento.

---

# 📋 Gobierno y cumplimiento

## 29 · Auditoría y Gestión de Riesgos
- [[Cálculo de riesgos (Probabilidad × Impacto)]] — escalas basadas en datos, matriz 5×5 y clasificación.
- [[Análisis de riesgos en proyectos de telecomunicaciones]] — categorías, PPDIOO, estrategias de tratamiento y registro de riesgos.
- [[Marcos normativos (NIST SP 800-53, Well-Architected y CSA CCM)]] — justificar controles, tabla de cumplimiento y recomendaciones.
- [[Plan de Respuesta a Incidentes (PRI)]] — NIST SP 800-61 / ISO 27035, severidad, contención y marco legal chileno.

---

# ⚔️ Ciberseguridad ofensiva

## 24 · Reconocimiento
- [[Reconocimiento Pasivo (OSINT)]] — Shodan, Wayback Machine, theHarvester, Google Dorks, GitHub OSINT, Certificate Transparency, buckets S3.
- [[Reconocimiento Activo de Red]] — Nmap, Masscan, ARP scan, Netdiscover, NBT-NS, LDAP, DNS Zone Transfer, Reconnoitre.
- [[Reconocimiento Web]] — subfinder, amass, ffuf, gobuster, katana, nuclei, httpx, detección de tecnologías y parámetros ocultos.

## 15 · Pentesting
- [[Nmap]] — descubrimiento de hosts, puertos, servicios y SO.
- [[Pentesting en Redes]] — Nmap, fuerza bruta SSH con Hydra y DoS con hping3.
- [[Fuzzing de Directorios y Subdominios]] — enumeración web con Gobuster, Feroxbuster y WFuzz.
- [[WPScan]] — auditoría de WordPress: core, plugins/temas vulnerables, usuarios y bruteforce de login.
- [[sqlmap]] — explotación automática de SQLi: enumeración, volcado de BD y shell.
- [[ARP Spoofing y MitM]] — envenenamiento ARP, DNS Spoofing y MitM con arpspoof/Bettercap.
- [[Metasploit y Payload Reverse TCP]] — payload con msfvenom y reverse shell Meterpreter.
- [[Metasploit — Módulos avanzados]] — workspaces, auxiliares, msfvenom multi-formato, gestión de sesiones, post-explotación completa y pivoting.
- [[Kali Linux — Herramientas (catálogo)]] — catálogo por fase: reconocimiento, explotación web, cracking, wireless, MitM, shells y wordlists.
- [[Laboratorio DVWA (CTF)]] — app web vulnerable (OWASP) en Docker para práctica.
- [[Laboratorio Kali Linux en Windows]] — montar Kali en WSL para un entorno de pentesting.

## 27 · Cracking de Contraseñas
- [[Proceso de Extracción de Hashes]] — scripts `*2john` (ssh/zip/pdf/keepass), `unshadow`, SAM/NTDS y capturas de red.
- [[Hashcat]] — cracking offline por GPU: modos de hash, diccionario, máscara, reglas e híbrido.
- [[John the Ripper]] — cracking offline multiformato en CPU, extracción `*2john` y autodetección.
- [[Hydra y Medusa]] — fuerza bruta **online** contra SSH, FTP, RDP, SMB y formularios web.

## 28 · Esteganografía
- [[Esteganografía y análisis de archivos]] — flujo de estegoanálisis: `file`/`exiftool`/`strings`/`binwalk`, Steghide, StegSeek y zsteg.

## 16 · Phishing
- [[GoPhish]] — framework de phishing simulado en Kali con MailHog como SMTP local.

## 17 · Monitoreo y rendimiento
- [[Splunk Enterprise]] — ingesta y análisis de logs en Ubuntu (base de SIEM).
- [[Zabbix y SNMP en Cisco]] — SNMPv2c en router Cisco y traps hacia Zabbix.
- [[Zabbix y SNMP en MikroTik]] — Zabbix Appliance y alta de un MikroTik por SNMP.
- [[Wireshark]] — análisis de tráfico; filtros de red y de seguridad más usados.
- [[iperf3 test de velocidad real]] — pruebas TCP/UDP, interpretación, metodología y laboratorio de enlace.
- [[Zabbix 7.4 con MySQL]] — instalación en Ubuntu 24.04 y despliegue en Docker.
- [[SNMPv3 en Cisco]] — usuarios, grupos, vistas, authPriv y alta en Zabbix.

---

# 📡 IoT, Cloud y Servicios

## 18 · IoT
- [[MQTT con Mosquitto en Debian]] — broker Mosquitto en Debian 12: instalación, pub/sub e integración con MySQL.
- [[Panel y comandos MQTT (Python)]] — shell remota sobre MQTT: intérprete y panel en Python.
- [[ESP8266 sketches y firmware]] — sketches Arduino (deauth, RFID, sensores a ThingsBoard) y firmware.
- [[Raspberry Pi pantalla OLED]] — IP/CPU/RAM y estado MQTT en OLED SSD1306 con luma.oled.
- [[ThingsBoard en Docker]] — plataforma IoT ThingsBoard con Docker.

## 19 · Meshtastic
- [[Meshtastic comandos y scripts]] — red mesh LoRa off-grid: comandos remotos y monitor OLED (Python).

## 20 · Cloud
- [[AWS - VPC, subredes, balanceo y autoescalado]] — VPC multi-AZ, EC2, AMI, ALB y Auto Scaling.
- [[MySQL + phpMyAdmin]] — MySQL en EC2 con conexión remota y administración por phpMyAdmin.
- [[Envío de mensajes MQTT desde PHP]] — interfaz web PHP que publica en un broker Mosquitto.

## 21 · Servicios
- [[PBX (Asterisk - FreePBX) en AWS]] — central IP en la nube desde la AMI de FreePBX.
- [[FreePBX 16 en VirtualBox]] — instalación desde ISO, extensiones PJSIP y softphones.
- [[Servidor IPTV con TVHeadend]] — servidor IPTV con listas m3u, consumido desde VLC.
- [[SQL (notas)]] — DDL/DML, consultas y enumeración de metadatos en RDBMS.

---

# ⚙️ Automatización y Programación

## 22 · Automatización
- [[Ansible]] — automatización agentless (SSH + Python): inventarios, playbooks, ad-hoc y módulos.

## 23 · Programación
- [[Servidor MQTT a MySQL (Python)]] — suscriptor MQTT que guarda mensajes de sensores en MySQL.
- [[Scripts SMI (consola y GUI)]] — herramientas SMIC para hablar por serial con nodos LoRa Mesh.

---

# 🔓 Seguridad Web

## 26 · Post-Explotación
- [[Escalada de Privilegios Linux]] — LinPEAS, SUID, capabilities, sudo, cron, NFS, docker/LXD, kernel exploits.
- [[Escalada de Privilegios Windows]] — WinPEAS, tokens (Potato), servicios, AlwaysInstallElevated, DLL hijacking, Pass-the-Hash.
- [[Persistencia Post-Explotación]] — crontab, systemd, SSH keys, registro, tareas programadas, reverse shells.
- [[Pivoting de Red]] — SSH tunneling, proxychains, Chisel, Ligolo-ng, netsh, graftcp.
- [[Active Directory — Ataques]] — enumeración (BloodHound), AS-REP/Kerberoasting, DCSync, Golden/Silver Ticket, NTLM Relay, abuso de ACLs y delegación.
- [[CrackMapExec - NetExec]] — reconocimiento SMB/LDAP/WinRM, spraying, Pass-the-Hash, volcado SAM/NTDS y ejecución remota en redes Windows/AD.

## 25 · Seguridad Web (OWASP / PayloadsAllTheThings)
> [!warning] Contenido para pruebas **autorizadas/educativas**. Ver [[📜 Fuentes y Licencias]].

**Inyección**
- [[Inyección SQL (SQLi)]] — union / blind / error-based, second-order y bypass de WAF.
- [[Inyección de comandos]] — ejecución de comandos del SO vía entrada no saneada.
- [[SSTI (Server-Side Template Injection)]] — Jinja2 / Twig / Freemarker hasta RCE.
- [[XXE (XML External Entity)]] — lectura de archivos, SSRF y OOB vía entidades XML.
- [[Inyección CRLF]] — HTTP response splitting, envenenamiento de cabeceras y de logs.
- [[Inyección LDAP]] — manipulación de filtros LDAP para bypass de login y fuga de atributos.
- [[Inyección NoSQL]] — operadores Mongo (`$ne`, `$gt`, `$regex`) para bypass y extracción.
- [[Inyección XSLT]] — transformaciones XSLT abusivas: lectura de archivos, SSRF y RCE.
- [[Inyección XPath]] — alterar consultas XPath sobre XML: bypass de login y blind.
- [[SSI (Server-Side Includes)]] — directivas `<!--#exec-->` hasta RCE; variante ESI.
- [[HTTP Parameter Pollution (HPP)]] — parámetros duplicados para burlar WAF y lógica.

**Cross-site y cliente**
- [[XSS (Cross-Site Scripting)]] — reflejado, almacenado, DOM y bypass de filtros.
- [[CSRF (Cross-Site Request Forgery)]] — forzar acciones y bypass de tokens.
- [[Clickjacking]] — iframe transparente para secuestrar clics; defensa con CSP/frame-busting.
- [[CORS mal configurado]] — política permisiva que filtra respuestas cross-origin con credenciales.
- [[Inyección CSS]] — CSS no confiable para exfiltración por selectores de atributo.
- [[WebSockets (seguridad)]] — CSWSH, falta de validación de origen y túnel de tráfico.
- [[DOM Clobbering]] — sobrescribir globales del DOM solo con HTML (sin JS).
- [[Tabnabbing]] — `window.opener` redirige la pestaña original a phishing.
- [[XS-Leaks]] — canales laterales cross-origin (timing, conteo de frames, errores).

**SSRF y redirección**
- [[SSRF (Server-Side Request Forgery)]] — acceso a interno / metadata cloud y bypass de filtros.
- [[HTTP Request Smuggling]] — desincronización front/back (CL.TE / TE.CL) para envenenar peticiones.
- [[Open Redirect]] — redirección a dominios externos; pivote a phishing y robo de tokens OAuth.
- [[Reverse Proxy mal configurado]] — path confusion, header smuggling y acceso a rutas internas.
- [[Web Cache Deception]] — engañar a la caché para que guarde datos privados del usuario.
- [[Virtual Hosts (vhost enumeration)]] — sitios ocultos por cabecera `Host` en una misma IP.

**Archivos y rutas**
- [[Inclusión de archivos (LFI-RFI)]] — wrappers `php://`, log poisoning, RCE.
- [[Path Traversal]] — lectura fuera de la raíz con `../` y encodings.
- [[Subida de archivos insegura]] — bypass de extensión / MIME / magic bytes.

**Lógica y acceso**
- [[IDOR]] — acceso a objetos ajenos por enumeración de identificadores.
- [[Deserialización insegura]] — gadgets en PHP / Java / Python (pickle) hacia RCE.
- [[Race Conditions]] — peticiones concurrentes para superar límites (limit-overrun, bypass de rate limit).
- [[Type Juggling (PHP)]] — comparaciones laxas (`==`) para bypass de login y hashes mágicos.
- [[ORM Leak]] — entrada del usuario como filtros del ORM: exfiltración campo a campo.
- [[Aleatoriedad insegura]] — tokens/IDs predecibles por PRNG débil o semilla temporal.
- [[Errores de lógica de negocio]] — abusar del flujo previsto (precios negativos, saltar pasos).
- [[Mass Assignment]] — asignar atributos no permitidos (`is_admin`) por autobinding.
- [[Parámetros ocultos]] — argumentos no documentados (`debug`, `admin`) que amplían la superficie.

**Autenticación y sesión**
- [[JWT (JSON Web Token)]] — `alg:none`, confusión de algoritmo, claves débiles y manipulación de claims.
- [[Ataques SAML]] — manipulación de aserciones, XSW y firma omitida para suplantación.
- [[OAuth mal configurado]] — `redirect_uri` laxa, falta de `state` y robo del `code`.
- [[Account Takeover]] — encadenar reset de contraseña, OAuth y sesión para apropiarse de cuentas.
- [[Fuerza bruta y rate limiting]] — credenciales/OTP y bypass del límite (IP rotation, carrera).

**API y dependencias**
- [[GraphQL (ataques)]] — introspección, batching, DoS por anidamiento e IDOR vía esquema.
- [[Fugas de API keys]] — secretos expuestos en JS, repos y apps; rotación y escaneo.

> [!note] En construcción
> Posibles fases siguientes: laboratorios CTF adicionales (HackTheBox / TryHackMe writeups), módulos de evasión de EDR/AV avanzados.

---

## 🔖 Etiquetas principales

`#cisco` · `#enrutamiento` · `#switching` · `#nat` · `#fhrp` · `#tacacs` · `#auditoria` · `#riesgos` · `#seguridad` · `#qos` · `#acl` · `#dhcp` · `#vlan` · `#ssh` · `#mikrotik` · `#fundamentos` · `#linux` · `#systemd` · `#docker` · `#criptografia` · `#firewall` · `#ids` · `#pentesting` · `#phishing` · `#monitoreo` · `#zabbix` · `#iot` · `#mqtt` · `#meshtastic` · `#cloud` · `#aws` · `#automatizacion` · `#programacion` · `#seguridad-web` · `#owasp` · `#sqli` · `#xss` · `#ssrf` · `#cracking` · `#fuerza-bruta` · `#esteganografia` · `#active-directory`
