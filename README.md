<div align="center">

# `> CEREBRO_VIRTUAL.cybersec`

### `// redes · cisco-ios · mikrotik · linux · pentesting · iot · cloud //`

![Obsidian](https://img.shields.io/badge/-Obsidian-0d0221?style=for-the-badge&logo=obsidian&logoColor=bd00ff)
![Cisco](https://img.shields.io/badge/-Cisco%20IOS-0d0221?style=for-the-badge&logo=cisco&logoColor=00ffff)
![MikroTik](https://img.shields.io/badge/-MikroTik-0d0221?style=for-the-badge&logo=mikrotik&logoColor=ff0080)
![Linux](https://img.shields.io/badge/-Linux-0d0221?style=for-the-badge&logo=linux&logoColor=bd00ff)
![Kali](https://img.shields.io/badge/-Kali%20Linux-0d0221?style=for-the-badge&logo=kalilinux&logoColor=00ffff)
![Docker](https://img.shields.io/badge/-Docker-0d0221?style=for-the-badge&logo=docker&logoColor=ff0080)
![AWS](https://img.shields.io/badge/-AWS-0d0221?style=for-the-badge&logo=amazonaws&logoColor=00ffff)
![Markdown](https://img.shields.io/badge/-Markdown-0d0221?style=for-the-badge&logo=markdown&logoColor=ffffff)

</div>

---

**Vault de Obsidian** con más de 150 notas de **redes, ciberseguridad ofensiva y defensiva, Linux, IoT y cloud**. Cubre desde configuración de equipos Cisco/MikroTik y enrutamiento dinámico hasta pentesting web (OWASP), Active Directory, post-explotación, pivoting, cracking de contraseñas, esteganografía, IDS/IPS, automatización con Ansible y despliegues en AWS. Cada nota está enlazada en un grafo de conocimiento navegable; los adjuntos (scripts, sketches, firmware) viven en `_adjuntos/`.

<!-- profile-excerpt -->
Vault Obsidian de **redes y ciberseguridad** — más de 150 notas en 30 secciones: **Cisco IOS / MikroTik**, administración **Linux**, **reconocimiento** (OSINT, Nmap, nuclei), **pentesting** (Metasploit avanzado, catálogo Kali, WPScan, sqlmap) y defensa (firewalls, **IDS/IPS**, criptografía, phishing, **TACACS+**), **auditoría y gestión de riesgos** (NIST 800-53, plan de respuesta a incidentes), **seguridad web** (45 vulns OWASP), **post-explotación** (escalada Linux/Windows, pivoting, **Active Directory**, CrackMapExec), **cracking** (Hashcat / John / Hydra) y **esteganografía**, **monitoreo** (Zabbix / Splunk / Wireshark), **IoT / Meshtastic**, **cloud** y servicios. Grafo de conocimiento navegable con nota índice **MOC** como cabeza del cerebro. `// net-codex · blue+red · knowledge-graph`
<!-- /profile-excerpt -->

> El punto de entrada del vault es la nota índice **`🗺️ MOC Ciberseguridad y Redes`**, que enlaza y organiza todo el conocimiento. El README queda fuera del grafo de Obsidian a propósito.

## `> tree ./`

| # | Sección | Contenido |
|---|---------|-----------|
| 00 | Fundamentos | GNS3 (laboratorios de red virtualizados, plantilla vIOS-L2), capacity planning, QoS, radioenlaces con RadioMobile |
| 01 | Cisco IOS — Operación | Config básica, CDP/LLDP/NTP, carga de IOS por ROMMON/TFTP, recuperación de contraseña |
| 02 | Direccionamiento y DHCP | Direcciones IP estáticas, rutas estáticas, servicio DHCP en Cisco, NAT y PAT |
| 03 | Enrutamiento dinámico | OSPF (wildcard, autenticación MD5/HMAC-SHA), EIGRP y redistribución EIGRP ↔ OSPF |
| 04 | Switching y VLAN | VLAN e Inter-VLAN, VTP/MVRP, STP/RSTP/MSTP, EtherChannel (LACP/PAgP), FHRP (HSRP/VRRP/GLBP) |
| 05 | ACL | Conceptos, ACL estándar y extendida |
| 06 | QoS | Voz sin switch administrable, QoS en router/switch con FreePBX |
| 07 | Seguridad Cisco | Capa 2, Port Security, DHCP Snooping, AAA, usuarios/privilegios, SSH, TACACS+ con tac_plus-ng |
| 08 | Conexiones remotas | SSH en Linux y en Raspberry Pi OS, Telnet en Cisco IOS |
| 09 | MikroTik | PPPoE para FTTH (montar un ISP) |
| 10 | Sistemas Operativos Linux | Inspección, red, procesos, almacenamiento, usuarios/permisos (sudoers por rol), systemd, logs (auth.log), seguridad (MFA en SSH), backups, scripting, virtualización, troubleshooting, red por distribución (netplan/nmcli) |
| 11 | Contenedores y Directorio Activo | Docker (comandos), unir Debian 12 a Active Directory |
| 12 | Criptografía | GPG (cifrado asimétrico y firmas), OpenSSL |
| 13 | Firewalls | iptables, pfSense en VirtualBox, portal cautivo en pfSense, FTDv en Cisco CML |
| 14 | IDS e IPS | Snort, Suricata (reglas propias, suricata-update, IPS con NFQUEUE, eve.json) |
| 15 | Pentesting | Nmap, Metasploit + Payload Reverse TCP, **Metasploit módulos avanzados** (workspaces, auxiliares, pivoting, evasión), **catálogo herramientas Kali** (9 fases: reconocimiento → post-explotación), WPScan, sqlmap, ARP Spoofing/MitM, fuzzing de directorios/subdominios, laboratorio DVWA, Kali en Windows |
| 16 | Phishing | GoPhish |
| 17 | Monitoreo y rendimiento | Wireshark, Splunk Enterprise, Zabbix 7.4, SNMPv3, Zabbix + SNMP en Cisco/MikroTik, iperf3 |
| 18 | IoT | ESP8266 sketches y firmware, MQTT con Mosquitto, panel MQTT en Python, Raspberry Pi OLED, ThingsBoard en Docker |
| 19 | Meshtastic | Comandos y scripts de red mesh LoRa |
| 20 | Cloud | AWS (VPC, subredes, balanceo, autoescalado), MQTT desde PHP, MySQL + phpMyAdmin |
| 21 | Servicios | FreePBX 16 en VirtualBox, PBX (Asterisk/FreePBX) en AWS, SQL (notas), servidor IPTV con TVHeadend |
| 22 | Automatización | Ansible |
| 23 | Programación | Scripts SMI (consola y GUI), servidor MQTT → MySQL en Python |
| 24 | Reconocimiento | OSINT pasivo (theHarvester, Shodan, FOFA), reconocimiento activo de red, reconocimiento web (subfinder, nuclei) |
| 25 | Seguridad Web | 45 vulns OWASP/PayloadsAllTheThings — inyección (SQLi, comandos, SSTI, XXE, CRLF, LDAP, NoSQL, XSLT, XPath, SSI, HPP), cross-site/cliente (XSS, CSRF, Clickjacking, CORS, CSS, WebSockets, DOM Clobbering, Tabnabbing, XS-Leaks), SSRF/redirección, archivos/rutas, lógica/acceso (IDOR, deserialización, race conditions, type juggling, ORM leak, mass assignment…), autenticación (JWT, SAML, OAuth, account takeover, fuerza bruta), API (GraphQL, fugas de API keys) |
| 26 | Post-Explotación | Escalada de privilegios Linux/Windows, persistencia, pivoting de red, Active Directory (ataques), CrackMapExec/NetExec |
| 27 | Cracking de Contraseñas | Extracción de hashes (`*2john`, `unshadow`, SAM/NTDS), Hashcat (GPU), John the Ripper (CPU/multiformato), fuerza bruta online con Hydra y Medusa |
| 28 | Esteganografía | Estegoanálisis de archivos (`file`, `exiftool`, `strings`, `binwalk`), Steghide, StegSeek y zsteg |
| 29 | Auditoría y Gestión de Riesgos | Cálculo de riesgos P × I, riesgos en proyectos de telecomunicaciones, NIST SP 800-53 / Well-Architected / CSA CCM, plan de respuesta a incidentes |

### `> ./_adjuntos/`

| Archivo | Descripción |
|---------|-------------|
| `esp8266-control-led-web-thingsboard.ino` | Control de LED vía web con ThingsBoard |
| `esp8266-monitor-paquetes-deauth.ino` | Monitor de paquetes deauth WiFi |
| `esp8266-mq2-dht11-thingsboard.ino` | Sensores MQ2 + DHT11 → ThingsBoard |
| `esp8266-rfid-rc522-pentest.ino` | RFID RC522 para pentesting físico |
| `esp8266_deauther_2.6.1_NODEMCU.bin` | Firmware deauther para NodeMCU |
| `smic_console.py` | Script SMI consola |
| `smic_console_gui.py` | Script SMI con interfaz gráfica |

## `> ./open.sh`

```bash
git clone https://github.com/PandaAkiraNakai/Cerebro-Virtual-Ciberseguridad.git
```

Abre la carpeta como vault en [Obsidian](https://obsidian.md) y arranca por la nota **`🗺️ MOC Ciberseguridad y Redes`**. Activa la vista de **grafo** para navegar las conexiones entre temas.

## `> ./sources.sh`

El vault reúne y reescribe material de estudio de las siguientes fuentes públicas. Atribución completa y aviso de uso ético en la nota interna `📜 Fuentes y Licencias`.

| Repo | Licencia | Secciones del vault |
|------|----------|---------------------|
| [swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | MIT | 24 · 25 · 26 (reconocimiento, seguridad web, post-explotación, AD) |
| [swisskyrepo/InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/) | MIT | 26 (escalada, pivoting, Active Directory) |
| [pulentoski/repositorio-configuraciones](https://github.com/pulentoski/repositorio-configuraciones) | — | 00–23 · 29 (redes, Cisco, Linux, IoT, cloud, auditoría) |
| [pulentoski/Herramientas-de-pentesting](https://github.com/pulentoski/Herramientas-de-pentesting) | — | 15 · 26 · 27 · 28 (WPScan, sqlmap, CrackMapExec, cracking, esteganografía) |
| [aw-junaid/Kali-Linux](https://github.com/aw-junaid/Kali-Linux) | MIT | Fase futura — catálogo de herramientas Kali |

> Todas las contraseñas, IPs y nombres de dominio en los ejemplos son ficticios / de laboratorio. Material con fines educativos.

---

<div align="center">

```
> exit 0 // jacked out
```

</div>
