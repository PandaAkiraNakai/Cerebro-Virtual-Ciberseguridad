---
tags:
  - firewall
  - iptables
  - linux
aliases:
  - iptables
  - netfilter
---

# iptables

**iptables** — herramienta de filtrado de paquetes para sistemas Linux que permite definir reglas para gestionar el tráfico de red, permitiendo o bloqueando paquetes según diversos criterios.

## Conceptos básicos

### Tablas

| Tabla | Propósito |
|---|---|
| `filter` | Filtrado de paquetes (predeterminada) |
| `nat` | Traducción de direcciones de red |
| `mangle` | Modificación de paquetes |
| `raw` | Manejo previo al procesamiento |

### Cadenas

| Cadena | Tráfico |
|---|---|
| `INPUT` | Dirigido al sistema |
| `OUTPUT` | Generado por el sistema |
| `FORWARD` | Que pasa a través del sistema |
| `PREROUTING` | Modificación previa al enrutamiento |
| `POSTROUTING` | Modificación posterior al enrutamiento |

### Acciones

`ACCEPT`, `DROP`, `REJECT`, entre otras.

## Comandos básicos

```bash
# Ver reglas
iptables -L -v -n

# Agregar una regla
iptables -A <cadena> -p <protocolo> --dport <puerto> -j <acción>

# Eliminar una regla
iptables -D <cadena> -p <protocolo> --dport <puerto> -j <acción>
```

### Guardar y restaurar

```bash
# Guardar (Debian/Ubuntu)
iptables-save > /etc/iptables/rules.v4

# Guardar (Red Hat/CentOS)
service iptables save

# Restaurar (Debian/Ubuntu)
iptables-restore < /etc/iptables/rules.v4
```

## Reglas básicas

```bash
# Permitir tráfico HTTP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Bloquear tráfico desde una IP específica
iptables -A INPUT -s <IP> -j DROP
```

## Seguridad avanzada

```bash
# Permitir SSH solo desde una IP confiable
iptables -A INPUT -p tcp --dport 22 -s <IP_confiable> -j ACCEPT

# Bloquear el resto del tráfico SSH no solicitado
iptables -A INPUT -p tcp --dport 22 -j DROP
```

## Mitigación de ataques

Protección contra DoS — limitar la tasa de conexiones SYN:

```bash
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 4 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

Frenar escaneos tipo nmap:

```bash
iptables -A INPUT -p tcp --syn -m limit --limit 10/min --limit-burst 20 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

Protección contra fuerza bruta a SSH — limitar intentos de conexión:

```bash
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP

# Bloquear una IP sospechosa manualmente
iptables -A INPUT -s <IP_sospechosa> -j DROP
```

## Registro y auditoría

```bash
# Registrar intentos de conexión SSH
iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH Attempt: "

# Ver reglas con detalle
iptables -L -v

# Contar paquetes y bytes con numeración de líneas
iptables -L -v -n --line-numbers
```

## Políticas predeterminadas

Modelo de denegación por defecto (deny-all): bloquea todo lo entrante y reenviado, permite lo saliente.

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

> [!warning] Cuidado con bloquearte a ti mismo
> Antes de aplicar `iptables -P INPUT DROP`, asegúrate de tener una regla que permita tu acceso SSH actual; de lo contrario perderás la conexión remota.

> [!tip] El orden importa
> iptables evalúa las reglas de arriba abajo y aplica la primera que coincide. Coloca las reglas de permiso específicas antes de las de bloqueo generales.

---
📎 Guías: ![[iptables-Introduccion.pdf]]

🔗 Relacionado: [[Portal cautivo en pfSense]] · [[Snort]] · [[Suricata]]
