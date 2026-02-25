---
title: Consultas SQL - Ejercicios con Usuarios
description: Colección de consultas SQL organizadas por niveles de dificultad
---

# Consultas SQL con la tabla `users`

---
Este documento contiene una serie de consultas SQL organizadas por niveles de dificultad, todas basadas en una tabla llamada `users`.

## Estructura de la tabla `users`

La tabla `users` contiene los siguientes campos:
- `id`: Identificador único
- `first_name`: Nombre
- `last_name`: Apellido
- `email`: Correo electrónico
- `birth_date`: Fecha de nacimiento
- `document_type`: Tipo de documento (CC, TI, etc.)
- `role`: Rol del usuario (admin, employee, etc.)
- `is_active`: Estado activo/inactivo
- `is_verified`: Verificación del usuario
- `marital_status`: Estado civil
- `children_count`: Número de hijos
- `company`: Empresa donde trabaja
- `monthly_income`: Ingreso mensual
- `profession`: Profesión
- `city`: Ciudad de residencia

---

## Nivel 1: Consultas Básicas

Consultas fundamentales de selección y filtrado simple.

### Listar todos los usuarios
```sql
SELECT * FROM users;
```

### Mostrar solo first_name, last_name, email
```sql
SELECT first_name, last_name, email FROM users;
```

### Filtrar usuarios cuyo role sea 'admin'
```sql
SELECT * FROM users WHERE role = 'admin';
```

### Filtrar usuarios con document_type = 'CC'
```sql
SELECT * FROM users WHERE document_type = 'CC';
```

### Mostrar usuarios mayores de 18 años
```sql
SELECT *
FROM users
WHERE TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18;
```

### Mostrar usuarios cuyo ingreso sea mayor a 5,000,000
```sql
SELECT * FROM users WHERE monthly_income > 5000000;
```

### Mostrar usuarios cuyo nombre empiece por "A"
```sql
SELECT * FROM users WHERE first_name LIKE 'a%';
```

### Mostrar usuarios que no tengan compañía
```sql
SELECT * FROM users WHERE company IS NULL;
```

---

## Nivel 2: Filtros Compuestos

Consultas con múltiples condiciones en el WHERE.

### Usuarios mayores de 25 años que sean 'employee'
```sql
SELECT * FROM users 
WHERE TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 25 
  AND role = 'employee';
```

### Usuarios con 'CC' que estén activos
```sql
SELECT * FROM users 
WHERE document_type = 'CC' 
  AND is_active = 1;
```

### Usuarios mayores de edad sin empleo
```sql
SELECT * FROM users 
WHERE TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18 
  AND role != 'employee';
```

### Usuarios con empleo y con ingresos mayores a 3,000,000
```sql
SELECT * FROM users 
WHERE role = 'employee' 
  AND monthly_income > 3000000;
```

### Usuarios casados con al menos 1 hijo
```sql
SELECT * FROM users 
WHERE marital_status = 'Casado' 
  AND children_count > 0;
```

### Usuarios entre 30 y 40 años
```sql
SELECT * FROM users 
WHERE TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 30 
  AND TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) <= 40;
```

### Usuarios 'admin' verificados mayores de 25 años
```sql
SELECT * FROM users 
WHERE role = 'admin' 
  AND TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 25 
  AND is_verified = 1;
```

---

## Nivel 3: Agregaciones Básicas

Consultas con funciones de agregación y GROUP BY.

### Contar usuarios por role
```sql
SELECT role, COUNT(*) AS total
FROM users
GROUP BY role;
```

### Contar usuarios por document_type
```sql
SELECT document_type, COUNT(*) AS total
FROM users
GROUP BY document_type;
```

### Contar cuántos usuarios están desempleados
-- Opción 1
```sql
SELECT COUNT(*) AS total_desempleados
FROM users
WHERE role != 'employee';
```

-- Opción 2
```sql
SELECT COUNT(role) AS total_desempleados
FROM users
WHERE role != 'employee';
```

### Calcular el promedio general de ingresos
```sql
SELECT AVG(monthly_income) AS promedio_ingresos
FROM users;
```

### Calcular el promedio de ingresos por role
```sql
SELECT role, AVG(monthly_income) AS promedio
FROM users
GROUP BY role;
```

---

## Nivel 4: Agrupaciones y Ordenamientos

Consultas con HAVING, ORDER BY y LIMIT.

### Mostrar profesiones con más de 20 personas
```sql
SELECT profession, COUNT(*) AS total
FROM users
GROUP BY profession
HAVING COUNT(*) > 20;
```

### Mostrar la ciudad con más usuarios
```sql
SELECT city, COUNT(*) AS total
FROM users
GROUP BY city
ORDER BY total DESC
LIMIT 1;
```

### Comparar cantidad de menores vs mayores de edad
```sql
SELECT
    SUM(TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18) AS mayores,
    SUM(TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) < 18) AS menores
FROM users;
```

### Promedio de ingresos por ciudad ordenado de mayor a menor
```sql
SELECT city, AVG(monthly_income) AS promedio
FROM users
GROUP BY city
ORDER BY promedio DESC;
```

### Mostrar las 5 personas con mayor ingreso
```sql
SELECT first_name, last_name, monthly_income
FROM users
ORDER BY monthly_income DESC
LIMIT 5;
```

---

## Nivel 5: Consultas Avanzadas

Consultas complejas con CASE, subconsultas y análisis más profundos.

### Clasificar usuarios por categoría de edad
-- Solo mostrar la clasificación sin modificar la tabla
```sql
SELECT 
    id,
    first_name,
    birth_date,
    CASE
        WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) < 18 THEN 'Menor'
        WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 18 AND 64 THEN 'Adulto'
        ELSE 'Adulto mayor'
    END AS categoria_edad
FROM users;
```

### Agregar columna de categoría de edad a la tabla
-- Agregar la columna
```sql
ALTER TABLE users ADD COLUMN categoria_edad VARCHAR(20);
```

-- Actualizar los valores
```sql
UPDATE users
SET categoria_edad = CASE
    WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) < 18 THEN 'Menor'
    WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 18 AND 64 THEN 'Adulto'
    ELSE 'Adulto mayor'
END;
```

### Contar usuarios por categoría de edad
```sql
SELECT 
    CASE
        WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) < 18 THEN 'Menor'
        WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 18 AND 64 THEN 'Adulto'
        ELSE 'Adulto mayor'
    END AS categoria_edad,
    COUNT(*) AS total_usuarios
FROM users
GROUP BY categoria_edad
ORDER BY total_usuarios DESC;
```

### Ranking de ingresos individual por ciudad
```sql
SELECT 
    city,
    monthly_income
FROM users
ORDER BY monthly_income DESC;
```

### Ranking de ingresos totales por ciudad
```sql
SELECT 
    city,
    SUM(monthly_income) AS total_ingresos
FROM users
GROUP BY city
ORDER BY total_ingresos DESC;
```

### Estadísticas completas por ciudad
```sql
SELECT 
    city,
    COUNT(*) AS total_usuarios,
    SUM(monthly_income) AS total_ingresos,
    AVG(monthly_income) AS ingreso_promedio
FROM users
GROUP BY city
ORDER BY total_ingresos DESC;
```

### Profesión con mayor ingreso promedio
```sql
SELECT 
    profession,
    AVG(monthly_income) AS ingreso_promedio
FROM users
GROUP BY profession
ORDER BY ingreso_promedio DESC
LIMIT 1;
```

### Usuarios con ingreso superior al promedio general
-- Todos los campos
```sql
SELECT *
FROM users
WHERE monthly_income > (
    SELECT AVG(monthly_income) 
    FROM users
);
```

-- Solo campos específicos
```sql
SELECT id, first_name, last_name, monthly_income
FROM users
WHERE monthly_income > (
    SELECT AVG(monthly_income) 
    FROM users
)
ORDER BY monthly_income DESC;
```

---

## Notas importantes

1. **Funciones de fecha**: Se utiliza `TIMESTAMPDIFF()` para calcular la edad de manera precisa.
2. **CURDATE()**: Obtiene la fecha actual del sistema.
3. **LIKE**: Para búsquedas con patrones de texto.
4. **GROUP BY**: Agrupa resultados para funciones agregadas.
5. **HAVING**: Filtra grupos después de la agrupación.
6. **Subconsultas**: Permiten usar resultados de una consulta dentro de otra.

---

## Ejemplos de uso práctico

Estas consultas pueden ser útiles para:

- **Análisis demográfico**: Distribución por edad, ciudad, profesión.
- **Análisis financiero**: Ingresos por segmento de usuarios.
- **Segmentación de usuarios**: Por roles, estado civil, nivel de ingresos.
- **Reportes gerenciales**: KPIs y métricas de negocio.