---
tags:
  - cisco
  - operación
  - recuperación
  - rommon
  - seguridad
aliases:
  - confreg 0x2142
  - Reset password Cisco
---

# Recuperación de contraseña en Cisco IOS mediante ROMMON

Procedimiento para eliminar la contraseña de un router Cisco vía **ROMMON** por acceso de consola.

## Pasos

1. **Interrumpir el arranque** con la señal Break (ver tabla en [[Carga de IOS por ROMMON y TFTP]]). Aparece `rommon 1 >`.

2. **Modificar el registro de configuración** para ignorar la NVRAM:

```
rommon 1 > confreg 0x2142
```

> `0x2142` arranca el router **ignorando la `startup-config`**.

3. **Reiniciar**:

```
rommon 2 > reset
```

4. **Borrar la configuración antigua**:

```cisco
Router> enable
Router# erase startup-config
```

5. **Restaurar el registro normal**:

```cisco
Router# configure terminal
Router(config)# config-register 0x2102
```

> `0x2102` indica cargar normalmente la `startup-config` desde NVRAM.

6. **Guardar y reiniciar**:

```cisco
Router(config)# exit
Router# reload
```

> [!success] Resultado
> El router inicia normalmente, **sin contraseñas**, con configuración vacía y registro `0x2102`.

> [!warning] Acceso físico = riesgo
> Esta técnica solo requiere consola física. Es la razón por la que el control de **acceso físico** a los equipos es parte de la seguridad de red.

---
🔗 Relacionado: [[Carga de IOS por ROMMON y TFTP]] · [[Acceso y autenticación segura en Cisco]]
