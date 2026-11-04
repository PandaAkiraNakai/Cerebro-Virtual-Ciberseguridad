---
tags:
  - cisco
  - nat
  - direccionamiento
aliases:
  - NAT
  - PAT
  - NAT Overload
  - Network Address Translation
---

# NAT y PAT en Cisco

**NAT (Network Address Translation)** traduce direcciones IP entre una red interna (privada) y una externa (pública). **PAT** (*NAT overload*) es la variante que permite que muchos equipos compartan una sola IP pública distinguiéndolos por puerto.

```text
192.168.10.10 ──── NAT ────► 203.0.113.10
```

Rangos privados (RFC 1918): `10.0.0.0/8` · `172.16.0.0/12` · `192.168.0.0/16`.

## Inside y outside

Antes de cualquier regla, marca qué interfaz mira a la LAN y cuál a Internet.

```text
        INTERNET
            │
     Gi0/1  │ OUTSIDE
         ┌──┴──┐
         │ R1  │
         └──┬──┘
     Gi0/0  │ INSIDE
     192.168.10.0/24
```

```cisco
interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat outside
```

## Tipos de NAT

| Tipo | Traducción | ACL | Pool | Usa puertos | Uso habitual |
|---|---|---|---|---|---|
| Estática | 1:1 permanente | No | No | No | Publicar servidores |
| Dinámica | 1:1 temporal | Sí | Sí | No | Repartir un pool público |
| PAT / Overload | Muchos:1 | Sí | Opcional | Sí | Salida a Internet |

### NAT estática

Relación fija entre una IP privada y una pública; no necesita ACL.

```cisco
ip nat inside source static 192.168.10.10 203.0.113.10
```

Para publicar **un solo servicio** (redirección de puerto) en vez de toda la IP:

```cisco
ip nat inside source static tcp 192.168.10.10 80 203.0.113.1 80
```

### NAT dinámica

```cisco
access-list 10 permit 192.168.10.0 0.0.0.255                        ! qué se traduce
ip nat pool PUBLIC_POOL 203.0.113.10 203.0.113.12 netmask 255.255.255.0
ip nat inside source list 10 pool PUBLIC_POOL
```

Si el pool se agota, los equipos siguientes quedan sin traducción hasta que se libere una dirección.

### PAT (overload)

```text
192.168.10.10:50001 ─┐
192.168.10.11:50002 ─┼──► 203.0.113.1:30001 / 30002 / 30003…
192.168.10.12:50003 ─┘
```

El router mantiene una tabla de traducciones para saber a qué equipo interno pertenece cada conexión. `overload` es lo que permite compartir la IP.

## Configuración de PAT paso a paso

Escenario: LAN `192.168.10.0/24`, router `192.168.10.1`, WAN `Gi0/1` con `203.0.113.1/30`.

```cisco
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
 no shutdown
interface GigabitEthernet0/1
 ip address 203.0.113.1 255.255.255.252
 ip nat outside
 no shutdown

access-list 10 permit 192.168.10.0 0.0.0.255
ip nat inside source list 10 interface GigabitEthernet0/1 overload

ip route 0.0.0.0 0.0.0.0 203.0.113.2     ! ruta por defecto hacia el ISP
```

> [!tip] El comando clave
> `ip nat inside source list <ACL> interface <WAN> overload`

Con ACL nombrada:

```cisco
ip access-list standard NAT-LAN
 permit 192.168.10.0 0.0.0.255
ip nat inside source list NAT-LAN interface GigabitEthernet0/1 overload
```

Para PAT basta una [[ACL estándar]] que identifique la red interna.

### PAT con pool

```cisco
ip nat pool PUBLIC_POOL 203.0.113.10 203.0.113.12 netmask 255.255.255.0
ip nat inside source list 10 pool PUBLIC_POOL overload
```

Con `interface` se usa la IP de la WAN; con `pool`, las direcciones del pool.

## Verificación y limpieza

```cisco
show ip nat translations           ! tabla de traducciones
show ip nat statistics             ! aciertos, fallos, interfaces inside/outside
show access-lists                  ! ¿la ACL tiene coincidencias?
show running-config | include ip nat
clear ip nat translation *         ! borra solo las traducciones dinámicas
```

## Errores frecuentes

| Síntoma | Causa probable |
|---|---|
| No aparecen traducciones | Falta `ip nat inside` u `ip nat outside`, o aún no hay tráfico |
| La ACL no tiene coincidencias | Red o wildcard mal escritas |
| Traduce pero no navega | Falta la ruta por defecto hacia el ISP |
| Sale pero no vuelve | No hay ruta de retorno hacia la red interna |

## Checklist PAT

- [ ] Identificar interfaces LAN y WAN
- [ ] `ip nat inside` y `ip nat outside`
- [ ] ACL de la red interna
- [ ] `ip nat inside source list … overload`
- [ ] Ruta por defecto
- [ ] Generar tráfico y revisar `show ip nat translations`

---
🔗 Relacionado: [[Direcciones IP estáticas y rutas estáticas]] · [[Servicio DHCP en Cisco]] · [[ACL estándar]] · [[PPPoE para FTTH (MikroTik)]]
