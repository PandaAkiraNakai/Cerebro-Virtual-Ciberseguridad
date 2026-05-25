---
tags:
  - cloud
  - aws
  - vpc
  - autoescalado
aliases:
  - AWS VPC y Auto Scaling
  - Infraestructura AWS
---

# AWS: VPC, subredes, balanceo y autoescalado

**Taller práctico para construir en AWS una red privada (VPC) con subredes públicas y privadas en dos zonas de disponibilidad, lanzar instancias EC2 con un servidor web, crear una AMI personalizada y desplegar un servicio web de alta disponibilidad con balanceador de carga (ALB) y autoescalado (Auto Scaling).**

## Arquitectura objetivo

- **VPC** con subredes **públicas** y **privadas** repartidas en 2 zonas de disponibilidad (alta disponibilidad).
- **NAT Gateway** en una subred pública para dar salida a internet a las instancias privadas sin exponerlas.
- **ALB** (Application Load Balancer) en las subredes públicas, repartiendo tráfico HTTP.
- **Auto Scaling Group** con instancias en las subredes privadas, escalando según uso de CPU.

## 1. Crear la VPC (VPC Wizard)

`VPC Dashboard → Launch VPC Wizard` → opción **VPC, subnets, etc.** Etiqueta `VPC-Lab`, CIDR IPv4 por defecto, 2 zonas de disponibilidad.

| Recurso | CIDR |
| :--- | :--- |
| Subred pública AZ-a | `10.0.10.0/24` |
| Subred pública AZ-c | `10.0.20.0/24` |
| Subred privada AZ-a | `10.0.100.0/24` |
| Subred privada AZ-c | `10.0.200.0/24` |

Crea **una** NAT Gateway (una sola AZ para ahorrar costos) y habilita **DNS hostnames** y **DNS resolution**.

> [!info] NAT Gateway
> Permite que las instancias de las subredes privadas inicien conexiones hacia internet (actualizaciones, descargas), pero impide que servicios externos inicien conexiones hacia ellas.

## 2. Endpoint de VPC para S3 (opcional)

`VPC → Endpoints → Create endpoint`, tipo **Gateway**, servicio **S3**, asociado a la `VPC-Lab` y a las **subredes privadas**. La ruta hacia S3 se agrega automáticamente a la tabla de enrutamiento privada.

> [!tip] Ventaja del endpoint
> El tráfico a S3 viaja dentro de la red de AWS (mayor seguridad/conformidad) y puede reducir costos frente a sacarlo por la NAT Gateway.

## 3. Lanzar instancia EC2 (servidor web base)

`EC2 → Launch instance`:

- **Tipo**: `t2.micro`.
- **Key pair**: *Proceed without a key pair*.
- **Red**: `VPC-Lab`, **subred pública**, *Auto-assign public IP* en **Enable**.
- **Security group**: permitir **HTTP (TCP/80)**.
- En **Advanced details → User data**, script de arranque que instala el stack LAMP y la app de ejemplo:

```bash
#!/bin/sh
# Instalar stack LAMP
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
yum -y install httpd php-mbstring

# Iniciar el servidor web
chkconfig httpd on
systemctl start httpd

# Desplegar las páginas de la app de laboratorio
if [ ! -f /var/www/html/immersion-day-app-php7.tar.gz ]; then
    cd /var/www/html
    wget https://<bucket-s3>/immersion-day-app-php7.tar.gz
    tar xvfz immersion-day-app-php7.tar.gz
fi

# Instalar el AWS SDK para PHP
if [ ! -f /var/www/html/aws.zip ]; then
    cd /var/www/html
    mkdir vendor && cd vendor
    wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.zip
    unzip aws.zip
fi

# Actualizar paquetes
yum -y update
```

Cuando la instancia esté en **Running**, accede por `http://<ip_publica>` (usa `http://`, no `https://`) para ver la página de prueba.

> [!info] User data
> Es un script de inicialización que se ejecuta solo la primera vez que arranca la instancia.

## 4. Crear una AMI personalizada

`EC2 → selecciona la instancia → Actions → Image and templates → Create Image`. Nombre `Web Server v1`. Esta "imagen dorada" se reutilizará para lanzar nuevas instancias idénticas.

Una vez creada la AMI, **termina** la instancia base (`Instance state → Terminate instance`): ya no es necesaria.

## 5. Application Load Balancer (ALB)

`EC2 → Load Balancers → Create → Application Load Balancer`:

- Nombre `Web-ALB`, en la `VPC-Lab`, sobre las **2 subredes públicas**.
- Security group nuevo `Web-ALB-SG` que permita **HTTP** desde **Anywhere-IPv4**.
- Crear **Target Group** `Web-TG` (sin instancias por ahora) y asociarlo al listener.

## 6. Plantilla de lanzamiento (Launch Template)

Primero, un security group `ASG-Web-Inst-SG` que permita **HTTP solo desde `Web-ALB-SG`** (así las instancias solo reciben tráfico vía el ALB):

| Clave | Valor |
| :--- | :--- |
| Tipo | HTTP |
| Origen | `Web-ALB-SG` |

Luego `EC2 → Launch Templates → Create`:

- AMI `Web Server v1`, tipo `t2.micro`, sin par de claves.
- Red: VPC, security group `ASG-Web-Inst-SG`.
- Etiqueta `Name = Web Instance`.
- IAM instance profile según el entorno (ej. `LabInstanceProfile`).

## 7. Auto Scaling Group

`EC2 → Auto Scaling Groups → Create`:

- Nombre `Web-ASG`, plantilla de lanzamiento `Web`.
- Red: `VPC-Lab`, **2 subredes privadas**.
- Balanceo: adjuntar al **target group `Web-TG`** existente; habilitar métricas de grupo en **CloudWatch**.
- Tamaño: **deseada 2, mínima 2, máxima 4**.
- Política: **Target tracking** sobre **utilización de CPU = 30 %**.

> [!info] Cómo escala
> Si el promedio de CPU baja del 30 % se reducen instancias; si lo supera, se agregan (hasta 4) y el ALB reparte la carga para volver al objetivo.

## 8. Verificación y prueba de carga

- Accede al **DNS name del ALB**: al recargar, el host que responde cambia entre zonas (Round Robin por defecto).
- Usa la opción **LOAD TEST** de la app de ejemplo para forzar CPU. En `Auto Scaling Groups → Monitoring/Activity` verás cómo se lanzan instancias adicionales (hasta 4) según la política.

> [!warning] Costos
> Recuerda detener la prueba de carga y eliminar los recursos (ASG, ALB, NAT Gateway, VPC) al terminar el laboratorio. La NAT Gateway y las instancias generan costos mientras estén activas.

---
📎 Guías: ![[AWS-VPC-Balanceo-Autoescalado.pdf]]

🔗 Relacionado: [[MySQL + phpMyAdmin]] · [[PBX (Asterisk - FreePBX) en AWS]] · [[Envío de mensajes MQTT desde PHP]]
