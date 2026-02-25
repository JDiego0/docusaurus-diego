# Ejercicio de Vistas MySQL

Base de datos `ejercicio_vistas` — 5 vistas sobre una tabla de usuarios con datos demográficos, financieros y de contacto.

---

## Estructura de la base de datos

### Tabla `users`

```sql
CREATE TABLE users (
    id               INT AUTO_INCREMENT PRIMARY KEY,
    first_name       VARCHAR(100) NOT NULL,
    last_name        VARCHAR(100) NOT NULL,
    birth_date       DATE         NOT NULL,
    gender           VARCHAR(20),
    marital_status   VARCHAR(30),
    education_level  VARCHAR(50),
    document_type    VARCHAR(20),
    document_number  VARCHAR(30),
    email            VARCHAR(100) UNIQUE,
    mobile           VARCHAR(20),
    phone            VARCHAR(20),
    address          VARCHAR(150),
    city             VARCHAR(80),
    state            VARCHAR(80),
    country          VARCHAR(80),
    profession       VARCHAR(100),
    company          VARCHAR(100),
    monthly_income   DECIMAL(15,2),
    CHECK (gender IN ('Masculino','Femenino','Otro'))
);
```

> La tabla incluye una restricción `CHECK` para validar que el campo `gender` solo acepte los valores `Masculino`, `Femenino` u `Otro`.

---

## Datos de prueba

15 registros de usuarios colombianos con perfiles variados (estudiantes, profesionales, adultos mayores).

| # | Nombre | Ciudad | Profesión | Ingreso mensual |
|---|--------|--------|-----------|-----------------|
| 1 | Carlos Ramirez | Bogotá | Ingeniero | $5,200,000 |
| 2 | Maria Lopez | Medellín | Médica | $8,500,000 |
| 3 | Juan Torres | Cali | Estudiante | $0 |
| 4 | Ana Gomez | Barranquilla | Contadora | $3,800,000 |
| 5 | Luis Herrera | Bogotá | Estudiante | — |
| 6 | Sofia Mendez | Bogotá | Abogada | $7,100,000 |
| 7 | Pedro Castro | Medellín | Arquitecto | $4,600,000 |
| 8 | Valentina Rios | Cali | Diseñadora | $2,900,000 |
| 9 | Andres Vargas | Bucaramanga | Electricista | $3,200,000 |
| 10 | Camila Pineda | Bogotá | Gerente | $12,000,000 |
| 11 | Ricardo Suarez | Medellín | Estudiante | $0 |
| 12 | Natalia Diaz | Cartagena | Enfermera | $3,500,000 |
| 13 | Felipe Mora | Cali | Programador | $4,200,000 |
| 14 | Isabella Reyes | Bogotá | Ama de casa | — |
| 15 | Sebastian Jimenez | Medellín | Consultor | $6,300,000 |

---

## Vista 1 — Usuarios mayores de edad
**`view_adult_users`**

Muestra únicamente los usuarios que tienen 18 años o más, calculando la edad con `TIMESTAMPDIFF`.

```sql
CREATE VIEW view_adult_users AS
SELECT
    id,
    first_name,
    last_name,
    document_type,
    document_number,
    city,
    country,
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age
FROM users
WHERE TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18;
```

> **`TIMESTAMPDIFF(YEAR, birth_date, CURDATE())`** calcula los años cumplidos entre la fecha de nacimiento y la fecha actual.

**Consulta de prueba:**
```sql
SELECT * FROM view_adult_users;
```

---

## Vista 2 — Contactos consolidados
**`view_user_contacts`**

Genera un directorio de contacto con nombre completo y número de teléfono disponible, priorizando celular sobre teléfono fijo.

```sql
CREATE VIEW view_user_contacts AS
SELECT
    CONCAT(first_name, ' ', last_name)        AS full_name,
    email,
    COALESCE(mobile, phone, 'Sin teléfono')   AS contact_number,
    address,
    city,
    state,
    country
FROM users;
```

> **`CONCAT`** une el nombre y apellido en un solo campo.  
> **`COALESCE`** devuelve el primer valor no nulo de la lista: si existe `mobile` lo usa, si no intenta `phone`, y si tampoco existe devuelve `'Sin teléfono'`.

**Consulta de prueba:**
```sql
SELECT * FROM view_user_contacts;
```

---

## Vista 3 — Usuarios con ingresos
**`view_users_with_income`**

Filtra únicamente los usuarios que tienen un ingreso mensual registrado y mayor a cero.

```sql
CREATE VIEW view_users_with_income AS
SELECT
    id,
    first_name,
    last_name,
    profession,
    monthly_income
FROM users
WHERE monthly_income IS NOT NULL
  AND monthly_income > 0;
```

> El filtro `WHERE monthly_income IS NOT NULL AND monthly_income > 0` excluye tanto los registros `NULL` como los estudiantes con ingreso `0`.

**Consulta ordenada por ingreso descendente:**
```sql
SELECT * FROM view_users_with_income
ORDER BY monthly_income DESC;
```

---

## Vista 4 — Resumen demográfico
**`view_demographic_summary`**

Vista orientada al análisis demográfico: combina edad calculada, género, estado civil, nivel educativo y ubicación.

```sql
CREATE VIEW view_demographic_summary AS
SELECT
    CONCAT(first_name, ' ', last_name)         AS full_name,
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age,
    gender,
    marital_status,
    education_level,
    city,
    country
FROM users;
```

**Consulta — cantidad de usuarios por ciudad:**
```sql
SELECT
    city,
    COUNT(*) AS total_users
FROM view_demographic_summary
GROUP BY city
ORDER BY total_users DESC;
```

---

## Vista 5 — Perfil ejecutivo
**`view_user_profile`**

Consolida la información profesional y económica de cada usuario en un perfil completo.

```sql
CREATE VIEW view_user_profile AS
SELECT
    CONCAT(first_name, ' ', last_name)         AS full_name,
    document_type,
    document_number,
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age,
    profession,
    education_level,
    company,
    monthly_income,
    city,
    country
FROM users;
```

**Consulta — usuarios con ingresos mayores a $3,000,000 ordenados por ciudad:**
```sql
SELECT * FROM view_user_profile
WHERE monthly_income > 3000000
ORDER BY city ASC;
```

---

## Resumen de vistas

| Vista | Propósito | Filtro principal |
|-------|-----------|-----------------|
| `view_adult_users` | Usuarios mayores de edad | `edad >= 18` |
| `view_user_contacts` | Directorio de contactos | Ninguno |
| `view_users_with_income` | Usuarios con ingresos | `income > 0` y `NOT NULL` |
| `view_demographic_summary` | Análisis demográfico | Ninguno |
| `view_user_profile` | Perfil ejecutivo completo | Ninguno |

---

## Funciones SQL utilizadas

| Función | Descripción |
|---------|-------------|
| `TIMESTAMPDIFF(YEAR, fecha, CURDATE())` | Calcula la edad en años |
| `CONCAT(valor1, ' ', valor2)` | Une cadenas de texto |
| `COALESCE(val1, val2, ..., default)` | Devuelve el primer valor no nulo |

---

*Ejercicio de práctica — MySQL Views.*
