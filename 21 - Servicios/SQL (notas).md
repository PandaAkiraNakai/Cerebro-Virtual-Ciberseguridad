---
tags:
  - servicios
  - sql
  - bases-de-datos
  - rdbms
aliases:
  - SQL
  - DDL y DML
  - Notas SQL
---

# SQL (notas)

**SQL (Structured Query Language) es el lenguaje estándar para administrar y manipular sistemas de gestión de bases de datos relacionales (RDBMS). Funciona con consultas declarativas: tú indicas qué quieres y el motor decide la ruta de ejecución más eficiente para recuperar, insertar o modificar registros.**

## Jerarquía de objetos en un RDBMS

- **Instancia / Servidor**: el proceso que gestiona el acceso a los datos.
- **Base de datos (Database)**: contenedor lógico que agrupa esquemas y objetos.
- **Esquema (Schema)**: espacio de nombres que agrupa tablas, vistas, procedimientos.
- **Tabla (Table)**: estructura de filas (registros) y columnas (campos).
- **Campo (Column)**: atributo con un tipo de dato (`INTEGER`, `VARCHAR`, etc.).

## Conectarse a una base de datos

```bash
mysql -h <host_o_ip> -u <usuario> -p
```

## DDL vs. DML

### DDL (Data Definition Language)
Define o modifica la **estructura** de los objetos (el "contenedor"):

- `CREATE`: crea objetos (bases de datos, tablas, vistas).
- `ALTER`: modifica la estructura de un objeto existente.
- `DROP`: elimina objetos de forma permanente.
- `TRUNCATE`: vacía una tabla pero mantiene su estructura.

### DML (Data Manipulation Language)
Gestiona los **datos** dentro de los objetos:

- `SELECT`: recupera datos (a veces clasificado como DQL).
- `INSERT`: añade registros.
- `UPDATE`: modifica registros existentes.
- `DELETE`: elimina registros.

| Característica | DDL (Definición) | DML (Manipulación) |
| :--- | :--- | :--- |
| Objetivo | Estructura / esquema | Datos / registros |
| Efecto | Cambia el diseño de la DB | Cambia el contenido de las filas |
| Analogía | Crear una carpeta (tabla) | Escribir en un archivo (registro) |
| Persistencia | Suele ser automática | Puede revertirse (rollback) si no hay commit |

## Operaciones fundamentales

### Acceso y enumeración de estructuras

| Acción | MySQL / MariaDB | PostgreSQL |
| :--- | :--- | :--- |
| Listar bases de datos | `SHOW DATABASES;` | `\l` |
| Seleccionar base de datos | `USE nombre_db;` | `\c nombre_db` |
| Listar tablas | `SHOW TABLES;` | `\dt` |
| Ver estructura de tabla | `DESCRIBE nombre_tabla;` | `\d nombre_tabla` |

### Consultas (DQL)

```sql
SELECT * FROM nombre_tabla;
SELECT columna1, columna2 FROM nombre_tabla;
SELECT * FROM nombre_tabla WHERE columna = 'valor';
SELECT * FROM nombre_tabla ORDER BY columna DESC;
```

### Inserción y manipulación (DML)

```sql
INSERT INTO nombre_tabla (columna1, columna2) VALUES ('dato1', 'dato2');
UPDATE nombre_tabla SET columna = 'nuevo_valor' WHERE id = 1;
DELETE FROM nombre_tabla WHERE id = 1;
```

## Enumeración de metadatos (information_schema)

Cuando no se conocen los nombres de los objetos, se consulta el **Information Schema** (estándar ANSI):

```sql
-- Listar tablas del esquema
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';

-- Listar columnas de una tabla
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'usuarios';
```

```sql
-- Versión del motor
SELECT @@version;     -- MySQL
SELECT version();     -- PostgreSQL
```

> [!info] Information Schema en auditoría
> La enumeración de `information_schema` es una técnica habitual tanto en administración como en pentesting de bases de datos (por ejemplo, tras una inyección SQL) para descubrir esquema, tablas y columnas.

> [!tip] Commit y rollback
> A diferencia del DDL, los cambios DML pueden revertirse con `ROLLBACK` mientras la transacción no se haya confirmado con `COMMIT`.

---
🔗 Relacionado: [[MySQL + phpMyAdmin]] · [[AWS - VPC, subredes, balanceo y autoescalado]]
