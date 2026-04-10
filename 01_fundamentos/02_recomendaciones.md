# Recomendaciones al escribir SQL

> Estas recomendaciones aplican principalmente a **SQL Server (T-SQL)**.  
> Donde existen diferencias importantes con otros motores, se indica explícitamente.

---

## 1. Palabras reservadas en MAYÚSCULAS

SQL es un lenguaje **case-insensitive** para sus palabras clave — `SELECT`, `select` y `SeLeCt` funcionan exactamente igual. Sin embargo, la convención profesional es clara:

✅ **MAYÚSCULAS** para palabras reservadas del lenguaje:

```sql
SELECT, FROM, WHERE, INSERT, UPDATE, DELETE,
CREATE, ALTER, DROP, JOIN, ON, GROUP BY,
ORDER BY, HAVING, INTO, VALUES, NULL, NOT NULL,
INT, VARCHAR, DATE, FLOAT, DECIMAL ...
```

✅ **minúsculas o snake_case** para nombres de objetos propios (tablas, columnas, alias):

```sql
-- ✅ Correcto
SELECT nombre_empleado, edad_empleado
FROM empleados
WHERE cargo = 'Analista';

-- ⚠️ Funciona, pero dificulta la lectura
select NOMBRE_EMPLEADO, EDAD_EMPLEADO
from EMPLEADOS
where CARGO = 'Analista';
```

> 💡 **¿Por qué importa?** No es solo estética — cuando otros lean tu código (o cuando tú lo leas 6 meses después), la distinción visual entre palabras del lenguaje y nombres de objetos hace el código mucho más fácil de entender.

---

## 2. Evitar espacios y caracteres especiales en nombres

Si un nombre de tabla o columna tiene espacios, SQL obliga a encerrarlo entre corchetes `[]` en SQL Server, o comillas graves `` ` `` en MySQL. Eso hace el código más lento de escribir y más propenso a errores.

```sql
-- ❌ Problemático
SELECT [nombre empleado], [fecha de contratacion]
FROM [empleados de rrhh]

-- ✅ Correcto
SELECT nombre_empleado, fecha_contratacion
FROM empleados_rrhh
```

### Convenciones de nomenclatura

| Estilo | Ejemplo | Recomendado para |
|--------|---------|-----------------|
| `snake_case` | `nombre_empleado` | Columnas y tablas ✅ |
| `PascalCase` | `NombreEmpleado` | Procedimientos almacenados |
| `camelCase` | `nombreEmpleado` | Evitar en SQL — más común en código |
| `MAYÚSCULAS` | `NOMBRE_EMPLEADO` | Evitar — dificulta lectura |

> **Recomendación personal:** usar siempre `snake_case` para tablas y columnas. Es el estándar más extendido y el más legible en SQL.

---

## 3. Indentación y formato del código

Un código bien indentado se lee como un libro. Uno mal indentado se convierte en un problema cuando hay que mantenerlo o compartirlo.

```sql
-- ❌ Incorrecto — todo en una línea
CREATE TABLE empleados (matricula INT PRIMARY KEY, nombre_empleado VARCHAR(100) NOT NULL, edad_empleado INT, cargo VARCHAR(50));

-- ✅ Correcto — cada elemento en su línea
CREATE TABLE empleados (
    matricula           INT             NOT NULL,
    nombre_empleado     VARCHAR(100)    NOT NULL,
    edad_empleado       INT,
    cargo               VARCHAR(50)
    CONSTRAINT pK_matricula_empleado PRIMARY KEY (matricula)
);
```

```sql
-- ❌ Consulta sin formato
SELECT nombre_empleado, cargo, salario FROM empleados WHERE salario > 1000000 AND cargo = 'Analista' ORDER BY salario DESC;

-- ✅ Consulta bien formateada
SELECT
    nombre_empleado,
    cargo,
    salario
FROM empleados
WHERE salario > 1000000
    AND cargo = 'Analista'
ORDER BY salario DESC;
```

> 💡 Una buena regla: **cada cláusula principal en su propia línea**, y las columnas/condiciones indentadas debajo.

---

## 4. Diferencias entre motores SQL

> ⚠️ Este curso está escrito en **T-SQL (SQL Server)**. Los conceptos son universales, pero la sintaxis tiene diferencias importantes según el motor que uses.

### 4.1 Números enteros

| Motor | Comportamiento |
|-------|---------------|
| **SQL Server** | `INT` soporta hasta 4 bytes (-2.147.483.648 a 2.147.483.647) |
| **MySQL / MariaDB** | Se puede especificar tamaño display: `INT(3)` — aunque no afecta el almacenamiento real |
| **PostgreSQL** | `INTEGER` e `INT` son equivalentes, igual rango que SQL Server |
| **SQLite** | No tiene tipos fijos — usa afinidad de tipos. `INTEGER` puede ser 1, 2, 4, 6 u 8 bytes según el valor |

### 4.2 Cadenas de texto

| Motor | VARCHAR | Texto largo |
|-------|---------|------------|
| **SQL Server** | `VARCHAR(n)` hasta 8.000 chars / `VARCHAR(MAX)` hasta 2GB | `TEXT` (obsoleto, usar VARCHAR(MAX)) |
| **MySQL** | `VARCHAR(n)` hasta 65.535 bytes | `TEXT`, `MEDIUMTEXT`, `LONGTEXT` |
| **PostgreSQL** | `VARCHAR(n)` o `TEXT` sin límite de tamaño | `TEXT` es equivalente a VARCHAR sin límite |
| **SQLite** | Acepta `VARCHAR(n)` pero lo trata como `TEXT` — el límite no se aplica | `TEXT` |

### 4.3 Fechas

| Motor | Solo fecha | Fecha + hora | Alta precisión |
|-------|-----------|-------------|---------------|
| **SQL Server** | `DATE` | `DATETIME` / `DATETIME2` | `DATETIME2(7)` |
| **MySQL** | `DATE` | `DATETIME` / `TIMESTAMP` | `DATETIME(6)` |
| **PostgreSQL** | `DATE` | `TIMESTAMP` | `TIMESTAMP WITH TIME ZONE` |
| **SQLite** | No tiene tipo DATE nativo | Almacena como `TEXT`, `REAL` o `INTEGER` | Depende del formato elegido |

> ⚠️ **SQLite y las fechas:** SQLite no tiene un tipo DATE real. Las fechas se guardan como texto (`'2024-02-12'`) y se manipulan con funciones específicas. Si venís de SQL Server, esto puede sorprender.

### 4.4 Funciones que cambian entre motores

| Operación | SQL Server | MySQL | PostgreSQL | SQLite |
|-----------|-----------|-------|-----------|--------|
| Fecha actual | `GETDATE()` | `NOW()` | `NOW()` | `DATE('now')` |
| Año de una fecha | `YEAR(fecha)` | `YEAR(fecha)` | `EXTRACT(YEAR FROM fecha)` | `strftime('%Y', fecha)` |
| Concatenar texto | `+` o `CONCAT()` | `CONCAT()` | `\|\|` o `CONCAT()` | `\|\|` |
| Top N filas | `TOP 10` | `LIMIT 10` | `LIMIT 10` | `LIMIT 10` |
| Convertir tipo | `CAST()` / `CONVERT()` | `CAST()` / `CONVERT()` | `CAST()` / `::tipo` | `CAST()` |
| Autoincremento | `IDENTITY(1,1)` | `AUTO_INCREMENT` | `SERIAL` / `GENERATED` | `AUTOINCREMENT` |

> 💡 **Regla práctica:** `CAST()` funciona en todos los motores. `CONVERT()` y `TOP` son exclusivos de SQL Server. `LIMIT` no existe en SQL Server — se usa `TOP`.

---

## 5. Comentarios en el código

Documentar el código es tan importante como escribirlo. En SQL hay dos formas:

```sql
-- Comentario de una línea (funciona en todos los motores)

/* Comentario
   de múltiples
   líneas */

-- Ejemplo aplicado:
SELECT
    nombre_empleado,
    salario,
    salario * 1.10 AS salario_con_aumento  -- +10% de aumento
FROM empleados
WHERE activo = 1  -- solo empleados activos
```

> 💡 **Cuándo comentar:** no expliques *qué* hace el código (eso se lee), explica *por qué* se hace así. Un comentario útil es `-- excluimos registros anteriores a 2020 por cambio de sistema`, no `-- filtramos por fecha`.

---

## 6. Resumen de buenas prácticas

| ✅ Hacer | ❌ Evitar |
|---------|---------|
| Palabras reservadas en MAYÚSCULAS | Mezclar mayúsculas y minúsculas sin criterio |
| `snake_case` para tablas y columnas | Espacios en nombres de objetos |
| Indentar cada cláusula en su línea | Consultas en una sola línea |
| Comentar el *por qué*, no el *qué* | Código sin ningún comentario |
| Usar `CAST()` para portabilidad | Depender de `CONVERT()` si el código puede migrar |
| Validar datos antes de insertar | Insertar sin verificar tipos y restricciones |

---

> 📁 Siguiente sección: [03 - Comandos DDL](./03_DDL.md)  
> 📁 Sección anterior: [01 - Tipos de datos](./01_tipos_de_datos.md)
