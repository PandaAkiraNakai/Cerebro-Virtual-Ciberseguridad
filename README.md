<div align="center">

# `> CEREBRO_VIRTUAL.cybersec`

### `// redes · cisco-ios · mikrotik · ssh · vault-obsidian //`

![Obsidian](https://img.shields.io/badge/-Obsidian-0d0221?style=for-the-badge&logo=obsidian&logoColor=bd00ff)
![Cisco](https://img.shields.io/badge/-Cisco%20IOS-0d0221?style=for-the-badge&logo=cisco&logoColor=00ffff)
![MikroTik](https://img.shields.io/badge/-MikroTik-0d0221?style=for-the-badge&logo=mikrotik&logoColor=ff0080)
![Markdown](https://img.shields.io/badge/-Markdown-0d0221?style=for-the-badge&logo=markdown&logoColor=ffffff)

</div>

---

**Vault de Obsidian** con apuntes de **redes y ciberseguridad**, centrado en equipos **Cisco IOS** y con material complementario de **MikroTik** y administración de **SSH en Linux**. Cada nota está enlazada en un grafo de conocimiento navegable; las guías originales en PDF viven en `_adjuntos/`.

> Este `README.md` es la portada del repositorio. El punto de entrada del vault es la nota índice **`🗺️ MOC Ciberseguridad y Redes`**, que enlaza y organiza todo el conocimiento (la cabeza del cerebro). El README queda fuera del grafo de Obsidian a propósito.

## `> tree ./`

| # | Sección | Contenido |
|---|---------|-----------|
| 00 | Fundamentos | Capacity planning, overhead, codecs y métricas de QoS |
| 01 | Cisco IOS — Operación | Config básica, CDP/LLDP/NTP, carga de IOS por ROMMON/TFTP, recuperación de contraseña |
| 02 | Direccionamiento y DHCP | Direcciones y rutas estáticas, servicio DHCP en Cisco |
| 03 | Enrutamiento dinámico | OSPF, EIGRP y redistribución EIGRP ↔ OSPF |
| 04 | Switching y VLAN | VLAN e Inter-VLAN, Spanning Tree (STP), EtherChannel con LACP |
| 05 | ACL | Conceptos, ACL estándar y extendida |
| 06 | QoS | Voz sin switch administrable, QoS en router/switch con FreePBX |
| 07 | Seguridad | Capa 2, Port Security, DHCP Snooping, AAA, usuarios/privilegios, SSH |
| 08 | Conexiones remotas | SSH en Linux y en Raspberry Pi OS |
| 09 | MikroTik | PPPoE para FTTH (montar un ISP) |

## `> ./open.sh`

```bash
git clone https://github.com/PandaAkiraNakai/Cerebro-Virtual-Ciberseguridad.git
```

Abre la carpeta como vault en [Obsidian](https://obsidian.md) y arranca por la nota **`🗺️ MOC Ciberseguridad y Redes`**. Activa la vista de **grafo** para navegar las conexiones entre temas.

## `> ./origen.sh`

Apuntes reorganizados y ampliados a partir del repositorio público [`pulentoski/repositorio-configuraciones` → `/Redes`](https://github.com/pulentoski/repositorio-configuraciones/tree/main/Redes). Material con fines educativos. Todas las contraseñas y direcciones de los ejemplos son ficticias / de laboratorio.

---

<div align="center">

```
> exit 0 // jacked out
```

</div>
