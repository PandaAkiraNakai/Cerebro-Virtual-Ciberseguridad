---
tags:
  - pentesting
  - phishing
  - ingenieria-social
  - gophish
aliases:
  - GoPhish
  - Gophish
---

# GoPhish — Framework de Phishing

**GoPhish** — framework de código abierto para simular campañas de phishing en evaluaciones de seguridad y concientización. Esta guía cubre su instalación en **Kali Linux** (VirtualBox) junto con **MailHog** como servidor SMTP local.

> [!warning] Solo en laboratorio autorizado
> Las campañas de phishing simulado deben hacerse con autorización explícita y sobre objetivos consentidos. Su uso fuera de ese marco es ilegal.

## Requisitos previos
- VM de Kali Linux en VirtualBox.
- Conexión a internet (solo para la descarga).
- Terminal con permisos de root o `sudo`.

## 1. Descargar y verificar

```bash
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip

# Verificar integridad SHA256
sha256sum gophish-v0.12.1-linux-64bit.zip
```

El hash debe coincidir exactamente con:

```text
44f598c1eeb72c3b08fa73d57049022d96cea2872283b87a73d21af78a2c6d47
```

> [!warning] Si el hash no coincide
> No continúes. Vuelve a descargar el archivo.

## 2. Descomprimir y dar permisos

```bash
unzip gophish-v0.12.1-linux-64bit.zip -d gophish
cd gophish
chmod +x gophish
```

## 3. Ejecutar GoPhish

```bash
sudo ./gophish
```

En el primer arranque genera una contraseña temporal en la terminal:

```text
level=info msg="Please login with the username admin and the password <CONTRASEÑA-GENERADA>"
```

> [!tip] Copia la contraseña
> Anótala antes de seguir; la necesitas para el primer login.

## 4. Acceder al panel

Abrir en el navegador de Kali:

```text
https://127.0.0.1:3333
```

El navegador mostrará una advertencia de certificado SSL: **Avanzado → Aceptar el riesgo y continuar**. Iniciar sesión con `admin` y la contraseña generada; el sistema obligará a cambiarla de inmediato.

## 5. Puertos que levanta GoPhish

| Servicio | Puerto | Uso |
|---|---|---|
| Panel de administración | 3333 (HTTPS) | Gestión de campañas |
| Servidor de phishing | 80 (HTTP) | Landing pages y tracking |

Verificar que ambos escuchan:

```bash
ss -tlnp | grep -E '80|3333'
```

## 6. MailHog (servidor SMTP local)

MailHog simula un servidor de correo sin necesidad de internet ni configuración externa.

```bash
sudo apt update
sudo apt install golang-go -y
go install github.com/mailhog/MailHog@latest
~/go/bin/MailHog
```

MailHog levanta:
- **SMTP** en el puerto `1025` (lo usa GoPhish para enviar).
- **Interfaz web** en `http://127.0.0.1:8025` (para ver los correos enviados).

## 7. Confirmar el entorno

Abrir dos terminales en Kali:
- **Terminal 1:** `sudo ./gophish`
- **Terminal 2:** `~/go/bin/MailHog`

| URL | Servicio |
|---|---|
| `https://127.0.0.1:3333` | Panel GoPhish |
| `http://127.0.0.1:8025` | Bandeja MailHog |

Si ambas cargan, el laboratorio está listo.

> [!info] Notas importantes
> - No cierres ninguna de las dos terminales mientras dure el laboratorio; cerrarlas detiene los servicios.
> - Si el puerto 80 da error de permisos, cambia el puerto de phishing en `config.json` a uno mayor a 1024 (ej. `8080`).
> - El archivo `gophish.db` guarda toda la configuración y resultados. No lo borres entre sesiones si quieres conservar el trabajo.

---
🔗 Relacionado: [[Laboratorio Kali Linux en Windows]] · [[ARP Spoofing y MitM]] · [[Laboratorio DVWA (CTF)]]
