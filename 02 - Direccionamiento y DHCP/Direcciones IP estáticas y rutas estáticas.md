---
tags:
  - cisco
  - enrutamiento
  - direccionamiento
aliases:
  - Rutas estáticas
  - IP estática Cisco
---

# Direcciones IP estáticas y rutas estáticas

## Asignar IP estática a una interfaz

```cisco
Router# configure terminal
Router(config)# interface Ethernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit
```

Verificar:

```cisco
Router# show ip interface brief
```

```
Interface     IP-Address    OK? Method Status Protocol
Ethernet0/0   192.168.1.1   YES manual up     up
Ethernet0/1   unassigned    YES unset  down   down
```

## Rutas estáticas

Entradas de tabla de enrutamiento configuradas **manualmente** para indicar cómo enrutar hacia destinos específicos. A diferencia de las rutas dinámicas ([[OSPF]], [[EIGRP]], RIP, BGP), las define el administrador.

**Utilidad:**
- **Control** preciso del tráfico.
- **Redundancia** como respaldo de rutas dinámicas.
- **Seguridad**: evitan propagación no autorizada de rutas.

```cisco
Router> enable
Router# configure terminal
Router(config)# ip route <red de destino> <máscara> <próximo salto>
```

> [!example] Topología de ejemplo
> Si **RT-1** no conoce `192.168.0.2/24` ni `10.10.2.0/24`, necesita dos rutas estáticas para alcanzarlas; lo mismo aplica a **RT-2** con `192.168.1.0/24` y `10.10.1.0/24`.

---
📎 Guía completa: ![[Config-Basica-Estatica-DHCP.pdf]]

🔗 Relacionado: [[Servicio DHCP en Cisco]] · [[OSPF]] · [[EIGRP]] · [[Configuración básica de equipos Cisco]]
