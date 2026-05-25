---
tags:
  - automatizacion
  - ansible
aliases:
  - Ansible
  - Automatización con Ansible
---

# Ansible

**Ansible es un motor de automatización declarativo y sin agente (agentless) que gestiona servidores mediante SSH y Python, definiendo el estado final deseado en lugar de los pasos para alcanzarlo.**

## ¿Qué es la automatización declarativa?

A diferencia de los scripts tradicionales (imperativos), Ansible es **declarativo**: defines el "estado final deseado" y la herramienta resuelve los pasos técnicos para alcanzarlo.

## Conceptos clave

- **Arquitectura agentless:** no requiere software residente en los nodos remotos. Utiliza **SSH** y **Python**, lo que minimiza el consumo de recursos y la superficie de ataque.
- **Idempotencia:** es la capacidad de ejecutar la misma operación varias veces sin cambiar el resultado tras la primera aplicación exitosa. Garantiza estabilidad y evita inconsistencias.
- **Nodo de control vs. nodo administrado:** el *Control Node* es la estación de trabajo donde reside Ansible (tu máquina local); los *Managed Nodes* son los servidores finales.

> [!info] La idempotencia es lo que distingue a Ansible de un simple script. Aplicar un playbook diez veces deja el sistema en el mismo estado que aplicarlo una sola vez.

## Instalación (nodo de control)

Para preparar el entorno de gestión en un sistema basado en Debian o Ubuntu:

```bash
# 1. Actualizar los índices de repositorios locales
sudo apt update

# 2. Instalar dependencias para gestionar repositorios PPA
sudo apt install software-properties-common -y

# 3. Incorporar el repositorio oficial de Ansible
sudo add-apt-repository --yes --update ppa:ansible/ansible

# 4. Instalar el motor de ejecución de Ansible
sudo apt install ansible -y
```

Verificación de la instalación:

```bash
ansible --version
```

## Estructura de archivos

Ansible requiere una estructura jerárquica de archivos para que la automatización funcione.

### El inventario (`hosts.ini`)

Es la base de datos lógica que define el alcance de la red y las direcciones de los nodos.

```ini
[servidores_iptv]
TU_HOST ansible_user=TU_USUARIO ansible_ssh_private_key_file=~/mis-llaves/TU_CLAVE.pem
```

> [!warning] No incluyas direcciones IP reales, usuarios ni rutas a claves privadas en archivos versionados. Usa variables o ficheros de inventario ignorados por Git.

### El playbook (`deploy.yml`)

Es el manual de estrategia escrito en YAML. Sus secciones principales:

- `hosts`: indica el grupo de servidores destino definido en el inventario.
- `become: yes`: indica que las tareas requieren escalada de privilegios (sudo).
- `tasks`: lista de acciones secuenciales que invocan módulos de Ansible.

## Comandos y ejemplos de uso

### Ejecución de playbooks (configuraciones complejas)

Procesa el archivo YAML y aplica todas las tareas definidas de forma secuencial.

```bash
ansible-playbook -i hosts.ini deploy.yml
ansible-playbook -i hosts.ini tvheadend.yml --private-key=TU_CLAVE.pem -u TU_USUARIO
```

### Comandos ad-hoc (gestión y auditoría en tiempo real)

Se utilizan para tareas de diagnóstico rápido sin escribir un playbook completo.

Verificar conectividad (módulo `ping`):

```bash
ansible servidores_iptv -i hosts.ini -m ping
```

Auditoría de memoria RAM (módulo `shell`):

```bash
ansible servidores_iptv -i hosts.ini -a "free -m"
```

Gestión de estados de servicio (módulo `service`):

```bash
ansible servidores_iptv -i hosts.ini -m service -a "name=tvheadend state=restarted" --become
```

## Módulos críticos en telecomunicaciones

| Módulo     | Función técnica                       | Aplicación en el sistema                          |
| ---------- | ------------------------------------- | ------------------------------------------------- |
| `apt`      | Gestión de paquetes `.deb`            | Instalación de dependencias base de Linux         |
| `snap`     | Gestión de paquetes autocontenidos    | Instalación aislada de TVHeadend                  |
| `get_url`  | Cliente de transferencia HTTP/HTTPS   | Descarga de listas M3U o archivos remotos         |
| `template` | Procesamiento de archivos Jinja2      | Despliegue de configuraciones dinámicas           |
| `ufw`      | Gestión de firewall (Netfilter)       | Apertura de puertos de streaming (9981, 9982)     |

## Ejemplo de diseño de tareas (YAML)

Un diseño profesional debe ser legible y registrar sus resultados para auditorías:

```yaml
- name: Instalación de TVHeadend mediante SNAP (método robusto)
  snap:
    name: tvheadend
    state: present
  register: resultado_log  # Almacena el resultado para verificar errores
```

## Resumen

Ansible transforma la gestión de infraestructuras en código fuente. La idea central es tratar la infraestructura como software: **versionable, auditable y automatizable**.

---
🔗 Relacionado: [[Servidor MQTT a MySQL (Python)]] · [[Scripts SMI (consola y GUI)]]
