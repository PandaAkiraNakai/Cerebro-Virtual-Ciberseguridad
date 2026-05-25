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

Mapa de contenidos del vault. Apuntes de **redes, sistemas y ciberseguridad**: equipos **Cisco IOS** y **MikroTik**, administración de **Linux**, **pentesting** y **seguridad defensiva** (firewalls, IDS/IPS, criptografía), **monitoreo** (Zabbix, Splunk, Wireshark), **IoT/Meshtastic**, **cloud** y **servicios**.

> [!info] Origen y fuentes
> Reorganizado y reescrito a partir de repositorios públicos. Atribución, licencias y aviso de uso ético en [[📜 Fuentes y Licencias]]. Las guías originales en PDF se conservan localmente y **no se publican** (privacidad); cada nota es autocontenida.

---

## 00 · Fundamentos
- [[Capacity Planning y QoS]] — dimensionamiento de ancho de banda, overhead, codecs y métricas de QoS.

## 01 · Cisco IOS — Operación
- [[Configuración básica de equipos Cisco]] — hostname, contraseñas, líneas, cifrado.
- [[Protocolos de descubrimiento y sincronización (CDP, LLDP, NTP)]]
- [[Carga de IOS por ROMMON y TFTP]] — recuperación de imagen `.bin`.
- [[Recuperación de contraseña por ROMMON]] — registro `0x2142`.
- [[Recuperación de equipos Cisco (referencia rápida)]]

## 02 · Direccionamiento y DHCP
- [[Direcciones IP estáticas y rutas estáticas]]
- [[Servicio DHCP en Cisco]]

## 03 · Enrutamiento dinámico
- [[OSPF]] — link-state, Dijkstra, áreas.
- [[EIGRP]] — híbrido, DUAL, AS.
- [[Redistribución EIGRP ↔ OSPF]]

## 04 · Switching y VLAN
- [[VLAN y enrutamiento Inter-VLAN]] — router-on-a-stick, dot1Q, trunk.
- [[Spanning Tree Protocol (STP)]] — RPVST, root bridge, PortFast, BPDU Guard.
- [[EtherChannel con LACP]] — agregación de enlaces.

## 05 · Listas de Control de Acceso (ACL)
- [[ACL - Conceptos]] — tipos, ubicación, dirección.
- [[ACL estándar]]
- [[ACL extendida]]

## 06 · Calidad de Servicio (QoS)
- [[QoS para voz sin switch administrable]]
- [[QoS en router y switch con FreePBX]]

## 07 · Seguridad
- [[Seguridad en la Capa 2]] — mapa de ataques y mitigaciones L2.
- [[Port Security]]
- [[DHCP Snooping]]
- [[AAA en Cisco]]
- [[Usuarios y niveles de privilegio]]
- [[SSH en Cisco]]
- [[Acceso y autenticación segura en Cisco]]

## 08 · Conexiones remotas
- [[SSH en Linux]]
- [[SSH en Raspberry Pi OS]]

## 09 · MikroTik
- [[PPPoE para FTTH (MikroTik)]] — crear un ISP / perfiles de navegación.

---

# 💻 Sistemas y administración

## 10 · Sistemas Operativos Linux
- [[01 - Inspección del sistema]] — kernel, hardware, CPU/RAM, dispositivos y módulos.
- [[02 - Red y conectividad]] — ip, ss, nmap, firewall, DNS y diagnóstico de red.
- [[03 - Procesos y rendimiento]] — top/htop, ps, señales, load average y OOM killer.
- [[04 - Almacenamiento y filesystems]] — df, particionado, LVM, SMART y fstab.
- [[05 - Usuarios, grupos y permisos]] — useradd, chmod, sudo, ACL y permisos especiales.
- [[06 - Servicios con systemd]] — systemctl, journalctl, targets, cron y timers.
- [[07 - Logs y auditoría]] — journalctl, grep/awk/sed, auditd y logrotate.
- [[08 - Seguridad del servidor]] — hardening SSH, fail2ban, UFW, Lynis y SSL/TLS.
- [[09 - Backups y recuperación]] — rsync, tar, dd, Borg, Restic y snapshots LVM.
- [[10 - Scripting y automatización]] — Bash: variables, bucles, funciones y manejo de errores.
- [[11 - Virtualización y contenedores]] — Proxmox (qm/pct), Docker, LXC y virsh.
- [[12 - Troubleshooting y diagnóstico]] — strace, tcpdump, lsof, perf e incidentes.

## 11 · Contenedores y Directorio Activo
- [[Comandos Docker]] — imágenes, contenedores, volúmenes, redes y Compose.
- [[Unir Debian 12 a Active Directory]] — integración con AD vía SSSD, Kerberos, PAM y Samba.

---

# 🛡️ Ciberseguridad

## 12 · Criptografía
- [[GPG - cifrado asimétrico y firmas]] — cifrado/descifrado, firmas digitales e intercambio de claves con GnuPG (OpenPGP).
- [[OpenSSL]] — cifrado de archivos, certificados SSL/TLS, hashing y referencia de algoritmos.

## 13 · Firewalls
- [[Portal cautivo en pfSense]] — autenticación de usuarios, certificado, DNS Resolver y Captive Portal.
- [[iptables]] — tablas, cadenas y reglas de filtrado en Linux; políticas por defecto y mitigación.

## 14 · IDS e IPS
- [[Snort]] — IDS/IPS: modos, reglas, preprocesadores e instalación en modo IPS.
- [[Suricata]] — IDS/IPS: instalación, `suricata.yaml`, reglas ET Open, logs y modos.

## 15 · Pentesting
- [[Nmap]] — descubrimiento de hosts, puertos, servicios y SO.
- [[Pentesting en Redes]] — Nmap, fuerza bruta SSH con Hydra y DoS con hping3.
- [[Fuzzing de Directorios y Subdominios]] — enumeración web con Gobuster, Feroxbuster y WFuzz.
- [[ARP Spoofing y MitM]] — envenenamiento ARP, DNS Spoofing y MitM con arpspoof/Bettercap.
- [[Metasploit y Payload Reverse TCP]] — payload con msfvenom y reverse shell Meterpreter.
- [[Laboratorio DVWA (CTF)]] — app web vulnerable (OWASP) en Docker para práctica.
- [[Laboratorio Kali Linux en Windows]] — montar Kali en WSL para un entorno de pentesting.

## 16 · Phishing
- [[GoPhish]] — framework de phishing simulado en Kali con MailHog como SMTP local.

## 17 · Monitoreo y rendimiento
- [[Splunk Enterprise]] — ingesta y análisis de logs en Ubuntu (base de SIEM).
- [[Zabbix y SNMP en Cisco]] — SNMPv2c en router Cisco y traps hacia Zabbix.
- [[Zabbix y SNMP en MikroTik]] — Zabbix Appliance y alta de un MikroTik por SNMP.
- [[Wireshark]] — análisis de tráfico; filtros de red y de seguridad más usados.
- [[iperf3 test de velocidad real]] — throughput, jitter y pérdida (cliente-servidor).

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

# 🔓 Seguridad ofensiva / Pentesting

## 25 · Seguridad Web (OWASP / PayloadsAllTheThings)
> [!warning] Contenido para pruebas **autorizadas/educativas**. Ver [[📜 Fuentes y Licencias]].

**Inyección**
- [[Inyección SQL (SQLi)]] — union / blind / error-based, second-order y bypass de WAF.
- [[Inyección de comandos]] — ejecución de comandos del SO vía entrada no saneada.
- [[SSTI (Server-Side Template Injection)]] — Jinja2 / Twig / Freemarker hasta RCE.
- [[XXE (XML External Entity)]] — lectura de archivos, SSRF y OOB vía entidades XML.

**Cross-site y cliente**
- [[XSS (Cross-Site Scripting)]] — reflejado, almacenado, DOM y bypass de filtros.
- [[CSRF (Cross-Site Request Forgery)]] — forzar acciones y bypass de tokens.

**SSRF y redirección**
- [[SSRF (Server-Side Request Forgery)]] — acceso a interno / metadata cloud y bypass de filtros.

**Archivos y rutas**
- [[Inclusión de archivos (LFI-RFI)]] — wrappers `php://`, log poisoning, RCE.
- [[Path Traversal]] — lectura fuera de la raíz con `../` y encodings.
- [[Subida de archivos insegura]] — bypass de extensión / MIME / magic bytes.

**Lógica y acceso**
- [[IDOR]] — acceso a objetos ajenos por enumeración de identificadores.
- [[Deserialización insegura]] — gadgets en PHP / Java / Python (pickle) hacia RCE.

> [!note] En construcción
> Próximas fases: resto de vulns web, reconocimiento, post-explotación y escalada, Active Directory, Metasploit y catálogo de herramientas de Kali.

---

## 🔖 Etiquetas principales
`#cisco` · `#enrutamiento` · `#switching` · `#seguridad` · `#qos` · `#acl` · `#dhcp` · `#vlan` · `#ssh` · `#mikrotik` · `#fundamentos` · `#linux` · `#systemd` · `#docker` · `#criptografia` · `#firewall` · `#ids` · `#pentesting` · `#phishing` · `#monitoreo` · `#zabbix` · `#iot` · `#mqtt` · `#meshtastic` · `#cloud` · `#aws` · `#automatizacion` · `#programacion` · `#seguridad-web` · `#owasp` · `#pentesting` · `#sqli` · `#xss` · `#ssrf`
