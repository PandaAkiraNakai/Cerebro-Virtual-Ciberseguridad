---
tags:
  - gns3
  - laboratorio
  - virtualizacion
  - redes
  - cisco
aliases:
  - GNS3
  - Laboratorios GNS3
  - Simulación de redes
---

# GNS3 — laboratorios de red virtualizados

**GNS3** (Graphical Network Simulator 3) monta topologías de red completas en un solo equipo. A diferencia de un simulador como Packet Tracer, **GNS3 no simula: emula** — ejecuta imágenes reales de Cisco IOS, máquinas virtuales y contenedores Docker, así que los comandos y el comportamiento son los del equipo auténtico.

## Arquitectura

| Componente | Función |
|---|---|
| `gns3` (GUI) | Interfaz gráfica; dibuja la topología y habla con el servidor por la API REST. |
| `gns3server` | Controlador + *compute*. Gestiona proyectos, nodos y enlaces. Escucha en `127.0.0.1:3080`. |
| `dynamips` | Emula routers Cisco clásicos (c3600, c7200) con imágenes IOS `.bin`. |
| `qemu`/KVM | Máquinas virtuales: vIOS / IOSvL2, pfSense, Windows, Linux. |
| `docker` | Contenedores para servicios y clientes Linux. |
| `vpcs` | Host virtual mínimo: solo IP, ping y traceroute. Consume casi nada. |
| `ubridge` | Puentea el tráfico entre nodos y con las interfaces del host. |

> [!info] El servidor es quien manda
> La GUI es solo un cliente. Si el servidor no está arriba, no hay laboratorio. Los nodos siguen corriendo aunque cierres la ventana.

## Instalación en Arch / CachyOS

```bash
yay -S gns3-gui gns3-server dynamips vpcs ubridge
sudo pacman -S qemu-full docker wireshark-qt
sudo systemctl enable --now docker
sudo usermod -aG docker,wireshark $USER
```

Arrancar el servidor cuando no lo lanza la GUI:

```bash
gns3server --local
```

> [!warning] `dynamips` no compila con GCC 16 y LTO
> Da un *internal compiler error*. Hay que reconstruirlo desde el `PKGBUILD` del AUR añadiendo `options=('!lto')` antes de `makepkg -si`.

## Elegir el tipo de nodo

| Nodo | Úsalo para | Coste |
|---|---|---|
| **dynamips** | Routers Cisco con IOS real (OSPF, EIGRP, ACL, DHCP, QoS). | ~512 MB por router |
| **qemu (vIOS-L2)** | Switches Cisco gestionables: VLAN, trunk, STP, port-security. | ~768 MB por switch |
| **docker** | Servidores y clientes Linux: FTP, DNS, web, Kali. | Casi nada |
| **VPCS** | Hosts de prueba para validar DHCP y conectividad. | Despreciable |
| **Cloud / NAT** | Sacar la topología a la red real o a Internet. | — |

### Contenedor o máquina virtual

Lo que se virtualiza es distinto: **una VM virtualiza el hardware** y arranca su propio kernel completo; **un contenedor virtualiza el sistema operativo** y comparte el kernel del host, aislando solo procesos, red y sistema de archivos.

| | Docker | VM (QEMU/KVM) |
|---|---|---|
| Arranque | Instantáneo | 30–60 s de boot |
| RAM | La del proceso | Reservada por completo |
| Kernel | El del host | Uno propio |
| SO invitado | Solo Linux | Cualquiera (IOS, Windows, BSD) |
| Aislamiento | Namespaces y cgroups | Hipervisor (más fuerte) |
| Persistencia | Solo rutas declaradas | Disco `qcow2` completo |

La regla práctica: **contenedor** para servicios Linux de los que quieras levantar muchos; **VM** cuando necesites otro kernel u otro sistema operativo — routers y switches Cisco, pfSense, Windows Server.

## Flujo de trabajo

1. **Plantillas** — importar la imagen una vez en *Edit → Preferences* (IOS routers, Qemu VMs, Docker containers). Después se arrastran al lienzo tantas veces como haga falta.
2. **Topología** — arrastrar nodos y unir puertos. En un switch qemu de 16 adaptadores, el adaptador *n* corresponde a `Gi0/n` en los cuatro primeros, y sigue por `Gi1/0`.
3. **Arrancar** los nodos (botón ▶ o por nodo, para no comerse la RAM de golpe).
4. **Consola** — doble clic abre telnet contra el puerto del nodo.

> [!tip] Idle-PC en dynamips
> Un router dynamips sin calibrar deja un núcleo al 100 %. Botón derecho sobre el router → *Idle-PC* → aplicar el valor con asterisco. Es lo primero que hay que hacer al importar una imagen IOS nueva.

## Consolas

Cada nodo escucha en un puerto telnet propio (5000, 5001, …), así que se puede trabajar sin la GUI:

```bash
telnet 127.0.0.1 5000     # R1
telnet 127.0.0.1 5002     # SW1
```

Comandos de un host **VPCS**:

```text
ip dhcp                   # pedir IP por DHCP (DORA)
ip 10.10.10.5/24 10.10.10.1   # IP estática + gateway
show ip                   # ver configuración
ping 10.10.40.10 -c 3
trace 10.10.40.10
save                      # guardar en startup.vpc
```

## Persistencia: dónde vive cada configuración

Es el punto que más problemas da. **Guardar el proyecto en la GUI no guarda la configuración de los equipos.**

| Nodo | Cómo se guarda | Dónde queda |
|---|---|---|
| Router IOS (dynamips) | `write memory` | `project-files/dynamips/<id>/configs/*_startup-config.cfg` |
| Switch vIOS (qemu) | `write memory` | Dentro del disco `hda_disk.qcow2` del nodo |
| VPCS | `save` | `project-files/vpcs/<id>/startup.vpc` |
| Docker | Solo `/etc/network/` y los volúmenes declarados | `project-files/docker/<id>/etc/network/interfaces` |

> [!warning] Lo que instales dentro de un contenedor se pierde
> Al reiniciar el nodo, GNS3 recrea el contenedor desde la imagen. Un `apk add` o un fichero suelto desaparecen. Para cambios permanentes hay que reconstruir la imagen con un `Dockerfile`.

**Snapshots**: *File → Snapshots* congela el proyecto entero (topología + discos + configuraciones), pero **exige el proyecto detenido**. Es el respaldo a hacer antes de tocar algo delicado.

## API REST v3 — automatizar el laboratorio

Todo lo que hace la GUI se puede hacer por API, útil para scripts de arranque y para configurar sin abrir ventanas:

```bash
# 1) Autenticación (obligatoria): devuelve un JWT
TOKEN=$(curl -s -X POST http://127.0.0.1:3080/v3/access/users/login \
  -d 'username=admin&password=admin' | jq -r .access_token)

# 2) Listar proyectos y nodos
curl -s -H "Authorization: Bearer $TOKEN" http://127.0.0.1:3080/v3/projects
curl -s -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:3080/v3/projects/$PROJECT/nodes | jq -r '.[] | "\(.name) \(.status) \(.console)"'

# 3) Arrancar o parar un nodo
curl -X POST -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:3080/v3/projects/$PROJECT/nodes/$NODE/start
```

La respuesta de cada nodo incluye su puerto de consola, con lo que se puede encadenar API + telnet para empujar configuraciones enteras desde un script.

## Capturar tráfico

Botón derecho sobre un **enlace** → *Start capture*. GNS3 abre Wireshark sobre ese cable virtual: sirve para ver el 802.1Q de un trunk, el intercambio DORA de DHCP o los *hello* de OSPF. Ver [[Wireshark]].

## Laboratorio de referencia

Topología típica de práctica, con todo lo anterior junto:

```text
   VLAN 10/20/69                 OSPF área 69              VLAN 30/40
  PCs + FTP ── SW1 ══trunk══ R1 ───────────── R2 ══trunk══ SW2 ── PCs
                             DHCP 10,20,69      DHCP 30,40
```

- **Inter-VLAN** por *router-on-a-stick*: una subinterfaz `.10`, `.20`, `.69` por VLAN — ver [[VLAN y enrutamiento Inter-VLAN]].
- **DHCP** en cada router para sus propias VLANs, con las IP de infraestructura excluidas — ver [[Servicio DHCP en Cisco]].
- **OSPF** en un área única sobre el enlace `/30` entre routers, con las subinterfaces de VLAN como `passive-interface` — ver [[OSPF]].
- **Port Security** en los puertos de acceso (`maximum 2`, `violation shutdown`) — ver [[Port Security]].
- **Servidor Linux** en contenedor Docker con IP fija dentro del rango excluido del pool.

## Verificación y problemas típicos

```cisco
show ip ospf neighbor        ! vecindad en FULL
show ip route ospf           ! rutas aprendidas
show ip dhcp binding         ! IP entregadas
show interfaces trunk        ! VLAN permitidas y nativa
show vlan brief
show port-security address   ! MAC seguras por puerto
```

| Síntoma | Causa habitual |
|---|---|
| El PC no coge IP | El puerto está en la VLAN equivocada, o la VLAN no está permitida en el trunk. |
| Puerto en `err-disabled` | Violación de port-security: se recupera con `shutdown` / `no shutdown`. |
| `Native VLAN mismatch` en el log | La VLAN nativa del trunk no coincide en los dos extremos, o no está creada en el switch. |
| El contenedor no tiene IPv4 | Falta configurar `/etc/network/interfaces` del nodo y reiniciarlo. |
| Un núcleo del host al 100 % | Falta calibrar el **Idle-PC** del router dynamips. |
| `401 Not authenticated` en la API | Falta el token: hay que llamar antes a `/v3/access/users/login`. |

---
🔗 Relacionado: [[VLAN y enrutamiento Inter-VLAN]] · [[OSPF]] · [[Servicio DHCP en Cisco]] · [[Port Security]] · [[Spanning Tree Protocol (STP)]] · [[Comandos Docker]] · [[Configuración básica de equipos Cisco]] · [[Wireshark]]
