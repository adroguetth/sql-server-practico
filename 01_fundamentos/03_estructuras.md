# Estructuras de las Consultas SQL

> Antes de aprender a filtrar, agrupar o unir tablas, es fundamental entender **cómo se construye una consulta SQL** — cuáles son sus partes, en qué orden se escriben, y en qué orden SQL las ejecuta realmente (que no es el mismo).

---

## 1. La estructura más básica: SELECT y FROM

Toda consulta SQL parte de dos cláusulas fundamentales. Sin ellas, no hay consulta:

```sql
SELECT columna1, columna2
FROM nombre_tabla;
```

- **`SELECT`** — le dice a SQL *qué quieres ver* (qué columnas)
- **`FROM`** — le dice a SQL *de dónde sacarlo* (qué tabla)

### Ejemplo

Supongamos que tenemos esta tabla `empleados`:

| matricula | nombre_empleado | cargo | salario | activo |
|-----------|----------------|-------|---------|--------|
| 1 | Ana Torres | Analista | 1.200.000 | 1 |
| 2 | Carlos Díaz | Gerente | 3.500.000 | 1 |
| 3 | María López | Coordinadora | 2.100.000 | 0 |

```sql
-- Traer solo nombre y cargo
SELECT nombre_empleado, cargo
FROM empleados;
```

**Resultado:**

| nombre_empleado | cargo |
|----------------|-------|
| Ana Torres | Analista |
| Carlos Díaz | Gerente |
| María López | Coordinadora |

```sql
-- Traer todas las columnas (usar con criterio, ver nota abajo)
SELECT *
FROM empleados;
```

> ⚠️ **Sobre el `SELECT *`:** Es útil para explorar una tabla rápidamente, pero en producción o en consultas finales **se recomienda listar las columnas explícitamente**. El `*` trae todo — incluso columnas que no necesitás, lo que puede ralentizar consultas en tablas grandes y hace el código más difícil de mantener.

---

## 2. La estructura completa de una consulta

Una consulta SQL puede tener muchas cláusulas. Esta es la estructura completa, en el **orden en que se escribe**:

```sql
SELECT      columnas o expresiones      -- Qué querés ver
FROM        tabla                       -- De dónde
JOIN        otra_tabla ON condicion     -- Unir con otra tabla (opcional)
WHERE       condicion                   -- Filtrar filas
GROUP BY    columna                     -- Agrupar (opcional)
HAVING      condicion_grupo             -- Filtrar grupos (opcional)
ORDER BY    columna ASC/DESC            -- Ordenar resultados (opcional)
OFFSET      n ROWS                      -- Paginación (opcional)
FETCH NEXT  n ROWS ONLY;               -- Paginación (opcional)
```

> 💡 No todas las cláusulas son obligatorias. Las únicas que **siempre deben estar** son `SELECT` y `FROM`.

### Ejemplo con varias cláusulas

```sql
SELECT
    nombre_empleado,
    cargo,
    salario
FROM empleados
WHERE activo = 1                    -- Solo empleados activos
    AND salario > 1000000           -- Con salario mayor a 1 millón
ORDER BY salario DESC;              -- Del mayor al menor salario
```

---

## 3. El orden de ejecución (cómo SQL realmente lee tu consulta)

Este es uno de los puntos más importantes y más ignorados en los cursos básicos.

**El orden en que escribís** la consulta **NO es el orden en que SQL la ejecuta.**

SQL procesa las cláusulas en este orden:

```
1. FROM          → Primero identifica la(s) tabla(s) de origen
2. JOIN          → Une las tablas si corresponde
3. WHERE         → Filtra las filas
4. GROUP BY      → Agrupa los resultados
5. HAVING        → Filtra los grupos
6. SELECT        → Selecciona las columnas a mostrar
7. ORDER BY      → Ordena el resultado final
8. OFFSET/FETCH  → Pagina el resultado
```

### ¿Por qué importa saber esto?

Porque explica varios errores comunes que confunden a quienes aprenden SQL:

**Error típico 1 — Usar un alias de SELECT en WHERE:**

```sql
-- ❌ Esto falla
SELECT salario * 1.10 AS salario_con_aumento
FROM empleados
WHERE salario_con_aumento > 1500000;  -- WHERE se ejecuta ANTES que SELECT
                                       -- el alias aún no existe

-- ✅ Correcto
SELECT salario * 1.10 AS salario_con_aumento
FROM empleados
WHERE salario * 1.10 > 1500000;       -- repetir la expresión
```

**Error típico 2 — Filtrar grupos con WHERE en vez de HAVING:**

```sql
-- ❌ Esto falla
SELECT cargo, COUNT(*) AS total
FROM empleados
WHERE total > 2          -- WHERE no puede filtrar resultados de COUNT()
GROUP BY cargo;          -- porque WHERE se ejecuta antes del GROUP BY

-- ✅ Correcto
SELECT cargo, COUNT(*) AS total
FROM empleados
GROUP BY cargo
HAVING COUNT(*) > 2;     -- HAVING filtra DESPUÉS de agrupar
```

---

## 4. Alias — renombrar columnas y tablas

Los alias hacen el resultado más legible y el código más limpio.

### Alias en columnas

```sql
-- Sin alias — el encabezado muestra la expresión completa
SELECT salario * 1.10
FROM empleados;

-- Con alias AS — el encabezado es legible
SELECT
    nombre_empleado,
    salario,
    salario * 1.10          AS salario_proyectado,
    GETDATE()               AS fecha_consulta
FROM empleados;
```

### Alias en tablas

Especialmente útil cuando hay JOINs o subconsultas:

```sql
-- Sin alias de tabla
SELECT empleados.nombre_empleado, cargo.nombre_cargo
FROM empleados
INNER JOIN cargo ON empleados.cargo = cargo.codigo_cargo;

-- Con alias de tabla — más limpio
SELECT e.nombre_empleado, c.nombre_cargo
FROM empleados   AS e
INNER JOIN cargo AS c ON e.cargo = c.codigo_cargo;
```

> 💡 En SQL Server (y en la mayoría de los motores), la palabra `AS` es opcional para los alias. `salario * 1.10 salario_proyectado` funciona igual que `salario * 1.10 AS salario_proyectado`. Sin embargo, **usar `AS` hace el código más legible** y es la convención recomendada.

---

## 5. SELECT con expresiones y cálculos

`SELECT` no solo trae columnas — también puede calcular valores al vuelo:

```sql
SELECT
    nombre_empleado,
    salario,
    salario * 0.10                          AS bono,
    salario + (salario * 0.10)              AS salario_total,
    GETDATE()                               AS fecha_consulta
FROM empleados;
```

> ⚠️ Estos cálculos **no modifican los datos en la tabla** — solo los muestran calculados en el resultado. Para modificar datos realmente, se usa `UPDATE`.

---

## 6. Diferencias entre motores

| Cláusula | SQL Server | MySQL | PostgreSQL | SQLite |
|----------|-----------|-------|-----------|--------|
| Limitar filas | `TOP 10` (en SELECT) | `LIMIT 10` (al final) | `LIMIT 10` (al final) | `LIMIT 10` (al final) |
| Fecha actual | `GETDATE()` | `NOW()` | `NOW()` | `DATE('now')` |
| Paginación | `OFFSET n ROWS FETCH NEXT n ROWS ONLY` | `LIMIT n OFFSET n` | `LIMIT n OFFSET n` | `LIMIT n OFFSET n` |
| Alias sin AS | ✅ Permitido | ✅ Permitido | ✅ Permitido | ✅ Permitido |

### Ejemplo: Traer los primeros 5 registros

```sql
-- SQL Server
SELECT TOP 5 nombre_empleado, salario
FROM empleados
ORDER BY salario DESC;

-- MySQL / PostgreSQL / SQLite
SELECT nombre_empleado, salario
FROM empleados
ORDER BY salario DESC
LIMIT 5;
```

---

## 7. Errores frecuentes

```sql
-- ❌ Error: olvidar el FROM
SELECT nombre_empleado;         -- ¿De qué tabla?

-- ❌ Error: escribir ORDER BY antes de WHERE
SELECT nombre_empleado
FROM empleados
ORDER BY salario DESC
WHERE activo = 1;               -- WHERE debe ir antes de ORDER BY

-- ❌ Error: usar * en producción sin criterio
SELECT *                        -- Trae 40 columnas cuando solo necesitás 3
FROM ventas;

-- ❌ Error: alias en WHERE
SELECT nombre_empleado, salario * 1.10 AS nuevo_salario
FROM empleados
WHERE nuevo_salario > 2000000;  -- El alias no existe aún en WHERE
```

---

## 8. Resumen visual del orden de escritura vs ejecución

```
ESCRITURA (orden que ves)     EJECUCIÓN (orden real de SQL)
─────────────────────────     ──────────────────────────────
1. SELECT                 →   1. FROM
2. FROM                   →   2. JOIN
3. JOIN                   →   3. WHERE
4. WHERE                  →   4. GROUP BY
5. GROUP BY               →   5. HAVING
6. HAVING                 →   6. SELECT  ← recién aquí existen los alias
7. ORDER BY               →   7. ORDER BY
```

> 💡 Tener este orden claro en la cabeza evita el 80% de los errores de sintaxis que cometen quienes aprenden SQL.

---

> 📁 Siguiente sección: [04 - Comandos DDL](./04_DDL.md)  
> 📁 Sección anterior: [02 - Recomendaciones](./02_recomendaciones.md)
