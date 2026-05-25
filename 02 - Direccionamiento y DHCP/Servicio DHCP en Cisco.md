---
tags:
  - cisco
  - dhcp
  - direccionamiento
aliases:
  - DHCP pool
  - ip dhcp pool
---

# Servicio DHCP en Cisco

**DHCP** (Protocolo de Configuración Dinámica de Host) automatiza la asignación de direcciones IP y otros parámetros de red a los clientes.

## Conceptos
1. **Cliente DHCP**: solicita IP y configuración.
2. **Servidor DHCP**: ofrece IPs y configuración.
3. **Proceso (DORA)**: el cliente *Discover* → el servidor *Offer* → el cliente *Request* → el servidor *Ack*.
4. **Configuración automática**: el cliente configura su interfaz con la IP asignada.

## Configuración del pool

```cisco
Router(config)# ip dhcp pool NOMBRE_POOL
Router(dhcp-config)# network DIRECCION_DE_RED MASCARA
Router(dhcp-config)# default-router DIRECCION_DEL_ROUTER
Router(dhcp-config)# dns-server DIRECCION_DNS
Router(dhcp-config)# exit
Router(config)# ip dhcp excluded-address DIR_INICIAL DIR_FINAL
```

## Ejemplo real

```cisco
Router(config)# ip dhcp pool MI_POOL_DHCP
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# lease 7
Router(dhcp-config)# exit
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
```

> [!tip] `ip dhcp excluded-address` reserva las IPs de infraestructura (gateway, servidores) para que el pool no las entregue.

> [!warning] Seguridad DHCP
> Un servidor DHCP no autorizado (*rogue*) puede secuestrar el tráfico. Protégete con [[DHCP Snooping]].

---
📎 Guías: ![[Servicio-DHCP-Cisco.pdf]]

🔗 Relacionado: [[Direcciones IP estáticas y rutas estáticas]] · [[DHCP Snooping]] · [[VLAN y enrutamiento Inter-VLAN]]
