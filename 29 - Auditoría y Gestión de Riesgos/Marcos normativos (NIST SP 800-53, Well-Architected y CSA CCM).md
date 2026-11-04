---
tags:
  - auditoria
  - normativa
  - nist
  - cloud
  - cumplimiento
aliases:
  - NIST SP 800-53
  - CSA CCM
  - Cloud Controls Matrix
  - Well-Architected
  - Zero Trust
---

# Marcos normativos: NIST SP 800-53, Well-Architected y CSA CCM

Configurar un control no basta: hay que poder decir **qué norma cumple y por qué**. Estos tres marcos sirven para justificar configuraciones, auditar y proponer mejoras.

## NIST SP 800-53 Rev. 5

Catálogo de controles de seguridad y privacidad del NIST (EE. UU.), agrupados en familias de dos letras. Es la referencia más usada para asociar cada configuración técnica a un control.

| Familia | Nombre | Tema |
|---|---|---|
| AC | Access Control | Quién accede y qué puede hacer |
| AU | Audit and Accountability | Registro y revisión de eventos |
| IA | Identification and Authentication | Cómo se comprueba la identidad |
| SC | System and Communications Protection | Red y cifrado |
| CM | Configuration Management | Configuración segura y mínima |

### Controles aplicados a un servidor Linux en la nube

| Control | Nombre | Implementación |
|---|---|---|
| AC-2 | Account Management | Usuarios y grupos por función (`useradd`, `groupadd`) |
| AC-3 | Access Enforcement | Permisos de archivos y reglas de sudoers |
| AC-5 | Separation of Duties | Un grupo administra y otro solo audita |
| AC-6 | Least Privilege | sudoers con comandos exactos, root deshabilitado |
| AC-7 | Unsuccessful Logon Attempts | `MaxAuthTries`, fail2ban |
| AC-12 | Session Termination | `TMOUT`, `LoginGraceTime` |
| AC-17 | Remote Access | SSH con llave, `AllowGroups`, `AllowUsers` por IP |
| AU-2 | Event Logging | Eventos en `auth.log` y en el registro del proveedor cloud (CloudTrail) |
| AU-3 | Content of Audit Records | Fecha, usuario, IP y resultado en cada evento |
| AU-6 | Audit Review, Analysis and Reporting | Revisión de `auth.log`, `last`, `lastb` y CloudTrail |
| AU-9 | Protection of Audit Information | El auditor no puede borrar logs |
| IA-2 | Identification and Authentication | Usuario único por persona, llave SSH |
| IA-2(1) | MFA para cuentas privilegiadas | TOTP en SSH |
| IA-5 | Authenticator Management | Llaves ED25519, permisos `600`/`400`, contraseña para sudo |
| SC-7 | Boundary Protection | Security Group con SSH solo desde una `/32` |
| SC-8 | Transmission Confidentiality and Integrity | SSH cifra la sesión |
| SC-28 | Protection of Information at Rest | Cifrado de volumen (EBS) o LUKS |
| CM-7 | Least Functionality | Solo los servicios y puertos necesarios |

Referencia: <https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final>

## Well-Architected: pilar de seguridad

Marco de buenas prácticas para cargas en la nube. Microsoft (Azure) y AWS tienen el suyo, ambos con un pilar de seguridad; sus principios aplican también a un servidor propio. El de Azure se apoya en **Zero Trust**.

| Principio Zero Trust | Significado | En el servidor |
|---|---|---|
| Verificar explícitamente | Autenticar y autorizar cada acceso | Llave SSH + MFA |
| Mínimo privilegio | Solo lo necesario, el tiempo necesario | sudoers por rol, `AllowGroups` |
| Asumir la brecha | Diseñar pensando que un control fallará | Registro, auditoría, cifrado, segmentación |

| Área | Aplicación |
|---|---|
| Identidades y accesos | Usuarios individuales, roles por grupo, MFA |
| Controles de red | Security Group restrictivo y firewall del sistema |
| Cifrado | En reposo (EBS/LUKS) y en tránsito (SSH) |
| Hardening | Root deshabilitado, sin contraseñas, servicios mínimos |
| Monitoreo y detección | Revisión de `auth.log` y CloudTrail |

Referencia: <https://learn.microsoft.com/es-es/azure/well-architected/security/>

## CSA Cloud Controls Matrix (CCM)

Marco de la Cloud Security Alliance con controles pensados para la nube, organizados en dominios y mapeados a ISO 27001, NIST y PCI DSS. Distingue qué le toca al proveedor y qué al cliente.

| Dominio | Nombre | Aplicación |
|---|---|---|
| IAM | Identity & Access Management | Usuarios, grupos, sudoers, MFA, separación de funciones |
| LOG | Logging and Monitoring | `auth.log`, CloudTrail, revisión de sesiones |
| CEK | Cryptography, Encryption & Key Management | Cifrado de volumen, llaves SSH |
| IVS | Infrastructure & Virtualization Security | Security Group, hardening de la instancia |
| TVM | Threat & Vulnerability Management | Parches de seguridad, Lynis |

Referencia: <https://cloudsecurityalliance.org/research/cloud-controls-matrix>

## Cómo justificar un control

Estructura: **configuración → riesgo que mitiga → controles que cumple**.

> [!example] Ejemplo
> Se configuró `PasswordAuthentication no` en SSH. Elimina la fuerza bruta contra contraseñas porque solo se aceptan llaves. Cumple NIST IA-2 e IA-5, el principio *verificar explícitamente* de Well-Architected y el dominio IAM de CSA CCM.

## Tabla de cumplimiento

| Control | Evidencia | Estado | Observación |
|---|---|---|---|
| AC-6 Least Privilege | `sudo -l -U auditor` muestra solo comandos de lectura | Cumple | — |
| IA-2(1) MFA | SSH pide `Verification code` | Cumple | — |
| AU-6 Audit Review | `auth.log` revisado; CloudTrail sin acceso | Parcial | La cuenta no tiene permisos sobre CloudTrail |
| AU-9 Protection of Audit Info | Logs solo locales | Parcial | Sin copia central |
| SC-28 Data at Rest | Volumen `Encrypted` | Cumple | — |

Estados: **Cumple**, **Parcial** o **No cumple**. Todo lo que no cumple completo genera una recomendación.

## Cómo redactar una recomendación

Estructura: **hallazgo → riesgo → recomendación → control**.

> [!example] Ejemplo
> **Hallazgo:** los logs se guardan solo en el servidor.
> **Riesgo:** un atacante con privilegios puede borrarlos y eliminar la evidencia.
> **Recomendación:** enviarlos a un servidor central con rsyslog o a un servicio de logs gestionado.
> **Control:** NIST AU-9 y dominio LOG de CSA CCM.

---
🔗 Relacionado: [[08 - Seguridad del servidor]] · [[05 - Usuarios, grupos y permisos]] · [[07 - Logs y auditoría]] · [[AWS - VPC, subredes, balanceo y autoescalado]] · [[Cálculo de riesgos (Probabilidad × Impacto)]] · [[Plan de Respuesta a Incidentes (PRI)]]
