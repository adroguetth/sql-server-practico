# Tipos de Datos en SQL Server

> **Nota:** Esta guía cubre los tipos de datos más usados en el día a día.  
> SQL Server tiene más tipos disponibles, pero los que se presentan aquí son los que realmente vas a necesitar en la práctica.

---

## 🔢 Números enteros

| Cláusula | Rango | Uso típico |
|----------|-------|------------|
| `INT` | -2,147,483,648 a 2,147,483,647 | IDs, cantidades, contadores |
| `BIGINT` | Números muy grandes | IDs de sistemas con millones de registros |
| `SMALLINT` | -32,768 a 32,767 | Códigos pequeños, edades |
| `TINYINT` | 0 a 255 | Flags, estados simples (0/1/2) |

> **Alias:** `INTEGER` es equivalente a `INT` en SQL Server.

```sql
-- Ejemplos
codigo_cargo    INT
edad_empleado   TINYINT
id_transaccion  BIGINT
```

---

## 🔣 Números decimales

| Cláusula | Descripción | Uso típico |
|----------|-------------|------------|
| `FLOAT` | Hasta 8 decimales, 32 bytes | Cálculos científicos, coordenadas |
| `DECIMAL(p, s)` | Precisión exacta definida por el usuario | Dinero, porcentajes, precios |
| `NUMERIC(p, s)` | Equivalente a `DECIMAL` | Igual que DECIMAL |

> ⚠️ **Recomendación:** Para valores monetarios, **preferir `DECIMAL` sobre `FLOAT`**.  
> `FLOAT` puede tener pequeñas imprecisiones por redondeo binario.  
> Ejemplo: `DECIMAL(10, 2)` almacena hasta 10 dígitos, con 2 decimales.

```sql
-- Ejemplos
precio          DECIMAL(10, 2)   -- ej: 1999.99
porcentaje      DECIMAL(5, 2)    -- ej: 98.75
coordenada      FLOAT            -- ej: -33.4569
```

---

## 🔤 Cadenas de texto

| Cláusula | Descripción | Uso típico |
|----------|-------------|------------|
| `VARCHAR(n)` | Texto de largo variable, hasta n caracteres | Nombres, descripciones, emails |
| `CHAR(n)` | Texto de largo fijo, siempre ocupa n caracteres | ⚠️ No recomendado (ver abajo) |

> ⚠️ **Recomendación: No usar `CHAR`.**  
> `CHAR(10)` siempre ocupa 10 caracteres aunque el valor tenga 3 — rellena con espacios.  
> `VARCHAR(10)` solo ocupa lo que necesita. Más eficiente en almacenamiento.

```sql
-- Ejemplos
nombre_empleado     VARCHAR(100)
descripcion_cargo   VARCHAR(255)
correo              VARCHAR(150)

-- Si se introduce una fecha entre comillas, se transforma a cadena de texto
fecha_texto         VARCHAR(20)    -- '12 de febrero'
estado              VARCHAR(20)    -- 'No registrado'
```

---

## 🌐 Caracteres especiales y Unicode

| Cláusula | Descripción | Uso típico |
|----------|-------------|------------|
| `NVARCHAR(n)` | Igual que VARCHAR pero soporta Unicode (tildes, ñ, emojis, caracteres asiáticos) | Direcciones, emails, textos con caracteres especiales |

> **Diferencia clave:** `VARCHAR` usa 1 byte por carácter. `NVARCHAR` usa 2 bytes — soporta cualquier carácter del mundo.

```sql
-- Ejemplos
direccion       NVARCHAR(200)    -- 'Avenida los Leones #023'
correo          NVARCHAR(150)    -- 'dlopez@gmail.com'
nombre          NVARCHAR(100)    -- 'José María Núñez'
```

---

## 📅 Fechas y horas

| Cláusula | Descripción | Ejemplo |
|----------|-------------|---------|
| `DATE` | Solo fecha | `2024-02-12` |
| `TIME` | Solo hora | `14:35:00` |
| `DATETIME` | Fecha + hora | `2024-02-12 14:35:00` |
| `DATETIME2` | Fecha + hora con mayor precisión | `2024-02-12 14:35:00.1234567` |

> **Recomendación:** Usar siempre el formato `YYYY-MM-DD` para fechas en SQL Server.  
> Es el formato estándar ISO y evita errores de interpretación regional.

```sql
-- Ejemplos
fecha_nacimiento    DATE           -- 2002-02-12
fecha_contratacion  DATETIME       -- 2024-03-21 09:00:00
hora_entrada        TIME           -- 08:30:00

-- Formatos que SQL Server acepta para DATE:
'2002-02-12'    -- ✅ Recomendado (ISO)
'12/02/2002'    -- ⚠️  Puede variar según configuración regional
'12 de febrero' -- ⚠️  Solo como VARCHAR, no como DATE
```

---

## ✅ Otros tipos útiles

| Cláusula | Descripción | Uso típico |
|----------|-------------|------------|
| `BIT` | Valor booleano: `0` (falso) o `1` (verdadero) | Flags de estado, activo/inactivo |
| `UNIQUEIDENTIFIER` | GUID (identificador único global) | IDs únicos entre sistemas |

```sql
-- Ejemplos
activo          BIT                  -- 1 = activo, 0 = inactivo
id_global       UNIQUEIDENTIFIER     -- '6F9619FF-8B86-D011-B42D-00C04FC964FF'
```

---

## 📋 Resumen rápido

| Necesito guardar... | Usar |
|---------------------|------|
| Un número entero (ID, cantidad) | `INT` |
| Un número con decimales (coordenada) | `FLOAT` |
| Un número con decimales exactos (precio, sueldo) | `DECIMAL(p, s)` |
| Texto normal | `VARCHAR(n)` |
| Texto con tildes, ñ, símbolos especiales | `NVARCHAR(n)` |
| Una fecha | `DATE` |
| La hora | `TIME` |
| Fecha y hora | `DATETIME` |
| Verdadero / Falso | `BIT` |

---

> 📁 Siguiente sección: [02 - Comandos DDL](./02_DDL.md)
