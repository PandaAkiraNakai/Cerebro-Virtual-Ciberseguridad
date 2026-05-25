---
tags:
  - firewall
  - pfsense
  - portal-cautivo
aliases:
  - Captive Portal pfSense
---

# Portal cautivo en pfSense

**Portal cautivo** — sistema que controla el acceso a la red y permite gestionar la velocidad de conexión. Se usa habitualmente en redes públicas para **autenticar a los usuarios** y **registrar su actividad**. Intercepta el tráfico y redirige al usuario a una página de inicio de sesión; solo tras autenticarse puede acceder a la red e Internet. Funciona en redes cableadas e inalámbricas.

## Beneficios

- **Identificación de usuarios:** rastrea quién accede a la red.
- **Registro y trazabilidad:** facilita el monitoreo del uso.
- **Versatilidad:** compatible con redes cableadas e inalámbricas.

## Configuración en pfSense

### 1. Usuarios locales

- Ve a **System / User Manager / Users**.
- Crea un usuario con nombre y contraseña para la autenticación.
- Crea un grupo y asígnale el privilegio de autenticarse en el portal cautivo.

### 2. Certificado autofirmado

- Ve a **System / Certificate Manager / CAs** y pulsa **Add**.

| Parámetro | Valor |
|---|---|
| Nombre descriptivo | Un nombre para el certificado |
| Método | *Create an internal Certificate Authority* |
| Longitud de clave | 2048 bits (por defecto) |
| Algoritmo de digestión | SHA-1 |
| Tiempo de vida | 365 días (o el deseado) |
| Nombre común | Dominio configurado (visible en el dashboard) |

Luego crea un **certificado de servidor** usando el CA recién creado.

> [!warning] Algoritmo de digestión
> La guía original indica SHA-1, pero está obsoleto. En despliegues nuevos prefiere **SHA-256** para el certificado.

### 3. DNS Resolver

- Ve a **Services / DNS Resolver / General Settings** y activa el **DNS Resolver**.
- En **Host Overrides**, pulsa **Add** y configura:

```text
Host:        nombre del host
Dominio:     nombre del dominio
Dirección IP: IP de la LAN
```

### 4. Portal cautivo

- Ve a **Services / Captive Portal** y pulsa **Add**.

| Parámetro | Función |
|---|---|
| Enable Captive Portal | Activa el portal |
| Interface | Interfaz de red a controlar |
| Hard Timeout | Tiempo máximo de conexión |
| Redirection URL | URL a la que se redirige tras autenticarse |
| MAC Filtering | Desactívalo para facilitar el acceso |
| Authentication | *Local* para usar usuarios de pfSense |
| Portal Page Contents | Personaliza la página de inicio de sesión |

También puedes habilitar el certificado configurado para una conexión segura (HTTPS).

### 5. Pruebas de funcionamiento

- Haz `ping` desde un cliente para verificar la conectividad.
- Al abrir un navegador, el portal redirige a la página de autenticación. Si el certificado no está instalado en el equipo cliente, aparecerá un aviso de seguridad.

## Seguridad y restricciones

> [!warning] Riesgo en la red local
> El portal cautivo bloquea el acceso directo a Internet, pero **no a la red local**, lo que puede habilitar ataques *Man-in-the-Middle*. Aísla el portal cautivo del resto de la red.

> [!tip] IDS recomendado
> Complementa el portal con un sistema de detección de intrusiones (IDS) como [[Snort]] o [[Suricata]] para mayor seguridad.

## Personalización adicional

- Carga plantillas HTML en **HTML Page Contents**.
- Sube archivos adicionales (imágenes, CSS) en **File Manager**.

---
📎 Guías: ![[Portal-Cautivo-pfSense.pdf]]

🔗 Relacionado: [[iptables]] · [[Snort]] · [[Suricata]]
