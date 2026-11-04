---
tags:
  - firewall
  - cisco
  - ftd
  - cml
  - laboratorio
aliases:
  - FTDv
  - Firepower Threat Defense
  - Cisco Secure Firewall
  - FDM
---

# FTDv en Cisco CML

**Cisco Secure Firewall Threat Defense Virtual** (FTDv) es el firewall de nueva generación de Cisco en formato virtual. En **CML** (Cisco Modeling Labs) se administra con la interfaz web local **FDM** (*Firepower Device Manager*), sin necesidad de un FMC central. Probado con FTD 7.7.0 y FX-OS 2.17.0.

## Procedimiento

1. **Agregar el nodo FTDv** desde el catálogo. No hace falta tocar el Day-0: trae IP y credenciales de fábrica.
2. **Cablear:**

```text
[ ext-conn-0 ]
      │ G0/0 (outside)
  [ ftdv-0 ]
      ├─ G0/1 ──────┐
      └─ Mgmt0/0 ───┤
               [ switch L2 ]
                    │
              [ host cliente ]
```

> [!warning] El único punto donde se falla
> **G0/1 y Mgmt0/0 van al mismo switch.** Si no, el cliente no alcanza la interfaz de gestión y el navegador no carga FDM.

3. **Esperar 4 a 6 minutos** de arranque. Antes de eso el login rechaza credenciales válidas.
4. **Entrar a FDM** desde el navegador del cliente en `https://192.168.45.45` con las credenciales por defecto del nodo de CML (usuario `admin`; revisa la contraseña en la documentación del nodo de tu versión). Acepta el certificado autofirmado. No se configura nada por consola.

## Direccionamiento de fábrica

| Interfaz | IP | Función |
|---|---|---|
| G0/0 | DHCP | outside |
| G0/1 | 192.168.45.1/24 | gateway de la LAN |
| Mgmt0/0 | **192.168.45.45/24** | FDM (web) |
| Pool DHCP | .46 a .254 | clientes |

> [!info] FDM vive en Mgmt0/0
> La `.1` responde ping pero no sirve el panel web.

## Easy Setup (primer login)

| Paso | Valor |
|---|---|
| Outside | IPv4 DHCP · IPv6 apagado |
| NTP | Zona local, servidores por defecto |
| Licencia | **Start 90-day evaluation** |

La evaluación cubre routing, NAT, ACL e interfaces; no cubre IPS, filtrado de URL ni antimalware. Al vencer, un *wipe* del nodo reinicia el contador. No refresques el navegador durante el asistente.

## Administración en FDM

| Sección | Contenido |
|---|---|
| Device → Interfaces | Interfaces, subinterfaces y VLAN |
| Device → Routing | Estático, OSPF, BGP |
| Policies → Access Control | Reglas de filtrado |
| Policies → NAT | Traducciones |
| Objects | Redes, puertos, URL |
| Monitoring → Events | Tráfico permitido y bloqueado |
| Troubleshoot → Packet Tracer | Simulación de un flujo |

> [!danger] Nada se aplica sin Deploy
> Cada cambio queda pendiente hasta pulsar **Deploy → Deploy Now**.

Política inicial:

| Regla | Acción |
|---|---|
| Trust Outbound Traffic | inside → outside permitido |
| Default Action | Bloquear todo lo demás |

## Diagnóstico

| Nodo cliente | Shell | Navegador |
|---|---|---|
| Desktop | Sí | Sí (**recomendado**) |
| Firefox | No | Sí |
| Alpine | Sí | No |

Cliente conectado a Mgmt0/0 sin DHCP:

```bash
sudo ip addr add 192.168.45.100/24 dev eth0
sudo ip link set eth0 up
```

---
🔗 Relacionado: [[pfSense en VirtualBox]] · [[Portal cautivo en pfSense]] · [[iptables]] · [[NAT y PAT en Cisco]] · [[GNS3 - laboratorios de red virtualizados]]
