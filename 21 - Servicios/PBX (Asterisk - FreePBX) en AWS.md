---
tags:
  - servicios
  - voip
  - freepbx
  - aws
aliases:
  - PBX en AWS
  - FreePBX en AWS
  - Asterisk en la nube
---

# PBX (Asterisk/FreePBX) en AWS

**Despliegue de una central telefónica IP (PBX) en AWS a partir de una AMI preconstruida de FreePBX (basado en Asterisk). Permite levantar una PBX en la nube lista para crear extensiones y troncales sin instalar Asterisk desde cero.**

## 1. Crear el par de claves

`EC2 → Pares de claves → Crear par de claves`:

- Nombre a elección.
- Tipo de par de claves: **RSA**.
- Formato de clave privada: **.pem**.

> [!warning] Guarda el .pem
> El archivo `.pem` se descarga una sola vez. Guárdalo en lugar seguro: es la llave SSH para administrar la instancia y no se puede volver a descargar.

## 2. Lanzar la instancia FreePBX

En el buscador de AMIs escribe `pbx` y selecciona la imagen **"FreePBX v16 with added packages and support"**. Lánzala asociando el par de claves creado en el paso anterior.

## 3. Configurar los puertos (Security Group)

Entra a la instancia y selecciona su **grupo de seguridad** asociado. Hay que abrir los puertos típicos de una PBX, tanto de **entrada** como de **salida**:

| Servicio | Protocolo / Puerto |
| :--- | :--- |
| Administración web | TCP/80, TCP/443 |
| SSH | TCP/22 |
| SIP (señalización) | UDP/5060 (y 5061 para TLS) |
| RTP (audio) | UDP/10000–20000 |

> [!warning] Exposición SIP
> Las PBX expuestas a internet son blanco habitual de ataques de fuerza bruta SIP y fraude telefónico. Restringe el origen de los puertos SIP/SSH a IPs de confianza, usa contraseñas fuertes en las extensiones y mantén `fail2ban` activo.

## 4. Acceso al panel de administración

Una vez creada la instancia y abiertos los puertos, accede por el navegador con la **IP pública** que entrega AWS.

En la opción **"Instrucciones de uso"** de la instancia hay un enlace del proveedor (TeleConnx) para activar la cuenta de administración, habilitar puertos y crear extensiones.

Credenciales por defecto del panel:

- usuario: `teleconnx`
- contraseña: **el ID de la instancia**

> [!tip] Primer paso tras entrar
> Cambia de inmediato la contraseña por defecto y crea tu propio usuario administrador. La contraseña inicial (ID de instancia) es predecible y no debe mantenerse.

> [!info] Documentación del proveedor
> Guía oficial de la AMI: `teleconnx.com/help/freepbx-aws-setup.html`.

---
📎 Guías: ![[PBX-Asterisk-FreePBX-AWS.pdf]]

🔗 Relacionado: [[AWS - VPC, subredes, balanceo y autoescalado]] · [[Servidor IPTV con TVHeadend]] · [[QoS en router y switch con FreePBX]]
