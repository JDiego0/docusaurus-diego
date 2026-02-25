# Normalización de Bases de Datos

---

## Ejercicio de normalización No.1

Este ejercicio modela un sistema de **ventas con clientes, productos y pedidos**, incluyendo la gestión de proveedores.

### Tablas

**clientes**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `last_name` VARCHAR(45)
- `email` VARCHAR(45)
- `ciudad_cliente_id` INT — FK hacia `ciudad_cliente`

**ciudad_cliente**
- `id` INT — Clave primaria
- `name` ENUM(...)

**pedidos**
- `id` INT — Clave primaria
- `fecha` DATE
- `clientes_id` INT — FK hacia `clientes`
- `quantity` INT
- `price` DECIMAL
- `total` DECIMAL

**productos**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `categoria_producto_id` INT — FK hacia `categoria_producto`

**categoria_producto**
- `id` INT — Clave primaria
- `name` ENUM(...)

**pedidos_has_productos** *(tabla intermedia — relación N:M entre pedidos y productos)*
- `pedidos_id` INT — FK hacia `pedidos`
- `pedidos_clientes_id` INT — FK hacia `clientes`
- `productos_id` INT — FK hacia `productos`

**proveedores**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `telefono` VARCHAR(45)

**proveedores_has_pedidos** *(tabla intermedia — relación N:M entre proveedores y pedidos)*
- `proveedores_id` INT — FK hacia `proveedores`
- `pedidos_id` INT — FK hacia `pedidos`
- `pedidos_clientes_id` INT — FK hacia `clientes`

### Relaciones
- Un cliente pertenece a una ciudad (`ciudad_cliente`).
- Un cliente puede tener muchos pedidos (1:N).
- Un pedido puede contener múltiples productos y un producto puede estar en múltiples pedidos (N:M → `pedidos_has_productos`).
- Un proveedor puede estar asociado a múltiples pedidos (N:M → `proveedores_has_pedidos`).

![Diagrama Ejercicio 1](../../static/img/numero%201.jpg)

---

## Ejercicio de normalización No.2

Este ejercicio modela un sistema de **operaciones comerciales entre personas y organizaciones**, con artículos categorizados por tipo.

### Tablas

**personas**
- `id` INT — Clave primaria
- `nombres` VARCHAR(45)
- `apellidos` VARCHAR(45)
- `correo` VARCHAR(45)
- `ciudades_id` INT — FK hacia `ciudades`

**ciudades**
- `id` INT — Clave primaria
- `name` ENUM(...)

**organizaciones**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `telefono` VARCHAR(45)

**operaciones**
- `id` INT — Clave primaria
- `referencia` VARCHAR(45)
- `fecha` DATE
- `cantidad` INT
- `precio` DECIMAL
- `total` DECIMAL
- `personas_id` INT — FK hacia `personas`
- `organizaciones_id` INT — FK hacia `organizaciones`

**articulos**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `tipo_articulo_id` INT — FK hacia `tipo_articulo`

**tipo_articulo**
- `id` INT — Clave primaria
- `name` ENUM(...)

**operaciones_has_articulos** *(tabla intermedia — relación N:M entre operaciones y artículos)*
- `operaciones_id` INT — FK hacia `operaciones`
- `operaciones_personas_id` INT — FK hacia `personas`
- `operaciones_organizaciones_id` INT — FK hacia `organizaciones`
- `articulos_id` INT — FK hacia `articulos`

### Relaciones
- Una persona pertenece a una ciudad (1:N).
- Una operación es realizada por una persona y una organización (N:1 en ambos casos).
- Una operación puede incluir múltiples artículos y un artículo puede pertenecer a múltiples operaciones (N:M → `operaciones_has_articulos`).
- Un artículo pertenece a un tipo de artículo (N:1).

![Diagrama Ejercicio 2](../../static/img/numero%202.jpg)

---

## Ejercicio de normalización No.3

Este ejercicio modela un sistema académico básico con **estudiantes, profesores y programas**.

### Tablas

**students**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `last_name` VARCHAR(45)
- `email` VARCHAR(45)

**teachers**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `last_name` VARCHAR(45)
- *(1 campo adicional no visible en el diagrama)*

**programs**
- `id` INT — Clave primaria
- `program` ENUM(...)

**teachers_has_students** *(tabla intermedia — relación N:M entre profesores y estudiantes)*
- `teachers_id` INT — FK hacia `teachers`
- `students_id` INT — FK hacia `students`

**students_has_programs** *(tabla intermedia — relación N:M entre estudiantes y programas)*
- `students_id` INT — FK hacia `students`
- `programs_id` INT — FK hacia `programs`

### Relaciones
- Un estudiante puede tener varios profesores y un profesor puede tener varios estudiantes (N:M → `teachers_has_students`).
- Un estudiante puede estar inscrito en varios programas y un programa puede tener varios estudiantes (N:M → `students_has_programs`).
- Un profesor puede estar asociado a un programa (relación opcional, representada con línea discontinua).

![Diagrama Ejercicio 3](../../static/img/numero%203.jpg)

---

## Ejercicio de normalización No.4

Este ejercicio amplía el modelo anterior con un sistema universitario más completo, incorporando **universidades, semestres y ciudades**.

### Tablas

**students**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `last_name` VARCHAR(45)
- `email` VARCHAR(45)

**teachers**
- `id` INT — Clave primaria
- `name` VARCHAR(45)
- `last_name` VARCHAR(45)
- `programs_id` INT — FK hacia `programs`
- `email` VARCHAR(45)

**programs**
- `id` INT — Clave primaria
- `program` ENUM(...)

**semestre**
- `id` INT — Clave primaria
- `no_semestre` ENUM(...)

**university**
- `id` INT — Clave primaria
- `name` ENUM(...)
- `ciudad_universidad_id` INT — FK hacia `ciudad_universidad`

**ciudad_universidad**
- `id` INT — Clave primaria
- `name` ENUM(...)

**teachers_has_students** *(tabla intermedia — relación N:M entre profesores y estudiantes)*
- `teachers_id` INT
- `students_id` INT

**students_has_programs** *(tabla intermedia — relación N:M entre estudiantes y programas)*
- `students_id` INT
- `programs_id` INT

**university_has_programs** *(tabla intermedia — relación N:M entre universidades y programas)*
- `university_id` INT
- `programs_id` INT

**programs_has_semestres** *(tabla intermedia — relación N:M entre programas y semestres)*
- `programs_id` INT
- `semestre_id` INT

### Relaciones
- Un estudiante puede estar con varios profesores (N:M → `teachers_has_students`).
- Un estudiante puede estar inscrito en varios programas (N:M → `students_has_programs`).
- Un programa puede ser ofrecido en múltiples universidades (N:M → `university_has_programs`).
- Un programa puede abarcar múltiples semestres (N:M → `programs_has_semestres`).
- Una universidad pertenece a una ciudad (`ciudad_universidad`).
- Un profesor tiene asignado un programa directamente (FK `programs_id` en `teachers`), con relación opcional a través de `university_has_programs`.

![Diagrama Ejercicio 4](../../static/img/numero%204.jpg)
