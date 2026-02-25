# 40 Consultas con express
---
Colección de 40 endpoints REST con sus respectivas consultas SQL para la gestión de usuarios, pedidos, productos y categorías.

## Link GitHub
---[Ir a Google](https://github.com/JDiego0/nodeexpress-40consultas)

## Consulta 1 — Usuario y sus pedidos
**`GET /api/usuario-pedidos/:nombre`**

Devuelve el nombre, correo y números de pedido de un usuario buscado por nombre.

```sql
SELECT
    users.name          AS nombre,
    users.email         AS correo,
    orders.order_number AS numero_pedido
FROM users
JOIN orders ON users.id = orders.user_id
WHERE users.name = ?
```

---

## Consulta 2 — Pedidos por correo electrónico
**`GET /api/pedidos-por-email/:email`**

Lista los pedidos de un usuario buscado por su dirección de email, ordenados del más reciente al más antiguo.

```sql
SELECT 
    users.name          AS nombre,
    orders.order_number AS codigo_pedido,
    orders.created_at   AS fecha
FROM users
JOIN orders ON users.id = orders.user_id
WHERE users.email = ?
ORDER BY orders.created_at DESC
```

---

## Consulta 3 — Productos con categoría
**`GET /api/productos-con-categoria`**

Lista todos los productos junto con la categoría a la que pertenecen, ordenados alfabéticamente.

```sql
SELECT products.name     AS producto,
       products.price    AS precio,
       categories.name   AS categoria
FROM products
JOIN categories ON products.category_id = categories.id
ORDER BY categories.name, products.name
```

---

## Consulta 4 — Usuarios sin compras
**`GET /api/usuarios-sin-compras`**

Devuelve los usuarios que no tienen ningún pedido registrado usando `LEFT JOIN`.

```sql
SELECT users.name  AS nombre,
       users.email AS correo
FROM users
LEFT JOIN orders ON users.id = orders.user_id
WHERE orders.id IS NULL
```

---

## Consulta 5 — Gasto total de un usuario
**`GET /api/gasto-total/:nombre`**

Calcula el gasto total acumulado de un usuario específico.

```sql
SELECT users.name                    AS nombre,
       ROUND(SUM(orders.total), 2)   AS gasto_total
FROM users
JOIN orders ON users.id = orders.user_id
WHERE users.name = ?
GROUP BY users.id, users.name
```

---

## Consulta 6 — Pedidos por estado
**`GET /api/pedidos-por-estado`**

Agrupa y cuenta los pedidos según su estado (`pending`, `cancelled`, etc.).

```sql
SELECT status        AS estado,
       COUNT(*)      AS total_pedidos
FROM orders
GROUP BY status
ORDER BY total_pedidos DESC
```

---

## Consulta 7 — Productos de Electrónica
**`GET /api/productos-electronica`**

Lista los productos de la categoría "Electrónica" ordenados por precio descendente.

```sql
SELECT products.name  AS producto,
       products.price AS precio
FROM products
JOIN categories ON products.category_id = categories.id
WHERE categories.name = 'Electrónica'
ORDER BY products.price DESC
```

---

## Consulta 8 — Detalle de una orden
**`GET /api/detalle-orden/:order_number`**

Devuelve los productos (id y cantidad) de una orden específica.

```sql
SELECT order_product.product_id AS id_producto,
       order_product.quantity   AS cantidad
FROM orders
JOIN order_product ON orders.id = order_product.order_id
WHERE orders.order_number = ?
```

---

## Consulta 9 — Usuarios por ciudad
**`GET /api/usuarios-ciudad/:ciudad`**

Lista los usuarios que han realizado pedidos desde una ciudad dada.

```sql
SELECT DISTINCT users.name  AS nombre,
                users.email AS correo
FROM users
JOIN orders ON users.id = orders.user_id
WHERE users.city = ?
```

---

## Consulta 10 — Promedio de pedidos por usuario
**`GET /api/promedio-pedidos-usuario`**

Calcula el promedio del total de pedidos por usuario, ordenado de mayor a menor.

```sql
SELECT users.name                        AS nombre,
       ROUND(AVG(orders.total), 2)       AS promedio_por_pedido
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name
ORDER BY promedio_por_pedido DESC
```

---

## Consulta 11 — Recibo de una orden
**`GET /api/recibo/:order_number`**

Genera el recibo completo de una orden: producto, precio vendido, cantidad y fecha.

```sql
SELECT orders.order_number        AS codigo_orden,
       orders.created_at          AS fecha,
       products.name              AS producto,
       order_product.price        AS precio_vendido,
       order_product.quantity     AS cantidad
FROM orders
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
WHERE orders.order_number = ?
```

---

## Consulta 12 — Ingresos por categoría
**`GET /api/ingresos-por-categoria`**

Suma los ingresos generados por cada categoría de productos.

```sql
SELECT categories.name                                              AS categoria,
       ROUND(SUM(order_product.price * order_product.quantity), 2) AS ingreso_total
FROM categories
JOIN products      ON products.category_id = categories.id
JOIN order_product ON order_product.product_id = products.id
GROUP BY categories.id, categories.name
ORDER BY ingreso_total DESC
```

---

## Consulta 13 — Productos comprados por un cliente
**`GET /api/productos-cliente/:nombre`**

Lista los productos únicos que ha comprado un cliente específico.

```sql
SELECT DISTINCT products.name AS producto
FROM users
JOIN orders        ON users.id = orders.user_id
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
WHERE users.name = ?
ORDER BY products.name
```

---

## Consulta 14 — Top 5 productos más vendidos
**`GET /api/top-productos`**

Devuelve los 5 productos con más unidades vendidas.

```sql
SELECT products.name                  AS producto,
       SUM(order_product.quantity)    AS unidades_vendidas
FROM products
JOIN order_product ON order_product.product_id = products.id
GROUP BY products.id, products.name
ORDER BY unidades_vendidas DESC
LIMIT 5
```

---

## Consulta 15 — Última venta por producto
**`GET /api/ultima-venta-productos`**

Muestra la fecha de la última venta registrada para cada producto.

```sql
SELECT products.name              AS producto,
       MAX(orders.created_at)     AS ultima_venta
FROM products
JOIN order_product ON order_product.product_id = products.id
JOIN orders        ON order_product.order_id = orders.id
GROUP BY products.id, products.name
ORDER BY ultima_venta DESC
```

---

## Consulta 16 — Compradores de productos Gamer
**`GET /api/compradores-gamer`**

Lista los usuarios que han comprado algún producto cuyo nombre contiene "Gamer".

```sql
SELECT DISTINCT users.name AS nombre
FROM users
JOIN orders        ON users.id = orders.user_id
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
WHERE products.name LIKE '%Gamer%'
```

---

## Consulta 17 — Ingresos por día
**`GET /api/ingresos-por-dia`**

Suma los ingresos diarios agrupando las órdenes por fecha.

```sql
SELECT DATE(orders.created_at)        AS dia,
       ROUND(SUM(orders.total), 2)    AS ingreso_dia
FROM orders
GROUP BY DATE(orders.created_at)
ORDER BY dia DESC
```

---

## Consulta 18 — Categorías sin ventas
**`GET /api/categorias-sin-ventas`**

Identifica categorías cuyos productos nunca han sido vendidos mediante `LEFT JOIN`.

```sql
SELECT categories.name AS categoria
FROM categories
LEFT JOIN products      ON products.category_id = categories.id
LEFT JOIN order_product ON order_product.product_id = products.id
WHERE order_product.id IS NULL
GROUP BY categories.id, categories.name
```

---

## Consulta 19 — Ticket promedio por usuario
**`GET /api/ticket-promedio-usuario`**

Muestra el número de órdenes y el ticket promedio de cada usuario.

```sql
SELECT users.name                    AS nombre,
       COUNT(orders.id)              AS num_ordenes,
       ROUND(AVG(orders.total), 2)   AS ticket_promedio
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name
ORDER BY ticket_promedio DESC
```

---

## Consulta 20 — Productos en órdenes canceladas
**`GET /api/productos-ordenes-canceladas`**

Lista los productos que aparecen en al menos una orden cancelada.

```sql
SELECT DISTINCT products.name AS producto
FROM orders
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
WHERE orders.status = 'cancelled'
ORDER BY products.name
```

---

## Consulta 21 — Reporte global
**`GET /api/reporte-global`**

Reporte completo que combina usuarios, órdenes, productos y categorías en una sola vista.

```sql
SELECT users.name                                                       AS usuario,
       users.city                                                       AS ciudad,
       orders.order_number                                              AS numero_orden,
       products.name                                                    AS producto,
       categories.name                                                  AS categoria,
       order_product.quantity                                           AS cantidad,
       ROUND(order_product.price * order_product.quantity, 2)          AS subtotal
FROM users
JOIN orders        ON users.id = orders.user_id
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
JOIN categories    ON products.category_id = categories.id
ORDER BY orders.order_number, products.name
```

---

## Consulta 22 — Ingresos de Ropa por ciudad
**`GET /api/ingresos-ropa-ciudad/:ciudad`**

Calcula los ingresos generados por la categoría "Ropa" en una ciudad específica.

```sql
SELECT users.city                                                       AS ciudad,
       ROUND(SUM(order_product.price * order_product.quantity), 2)     AS ingresos_ropa
FROM users
JOIN orders        ON users.id = orders.user_id
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
JOIN categories    ON products.category_id = categories.id
WHERE categories.name = 'Ropa' AND users.city = ?
GROUP BY users.city
```

---

## Consulta 23 — Cliente del año
**`GET /api/cliente-del-anio`**

Devuelve el cliente con el mayor gasto total acumulado.

```sql
SELECT users.name                    AS nombre,
       users.email                   AS correo,
       ROUND(SUM(orders.total), 2)   AS gasto_total
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name, users.email
ORDER BY gasto_total DESC
LIMIT 1
```

---

## Consulta 24 — Productos sin ventas
**`GET /api/productos-sin-ventas`**

Lista productos que nunca han sido comprados.

```sql
SELECT products.name  AS producto,
       products.price AS precio
FROM products
LEFT JOIN order_product ON order_product.product_id = products.id
WHERE order_product.id IS NULL
ORDER BY products.name
```

---

## Consulta 25 — Ganancia real por producto
**`GET /api/ganancia-real`**

Calcula la ganancia por unidad y total para cada producto vendido, comparando precio de venta contra costo.

```sql
SELECT products.name                                                         AS producto,
       ROUND(order_product.price - products.cost, 2)                        AS ganancia_por_unidad,
       order_product.quantity                                                AS unidades,
       ROUND((order_product.price - products.cost) * order_product.quantity, 2) AS ganancia_total
FROM products
JOIN order_product ON order_product.product_id = products.id
ORDER BY ganancia_total DESC
```

---

## Consulta 26 — Compradores de Videojuegos que no compraron Hogar
**`GET /api/compradores-videojuegos-no-hogar`**

Usuarios que compraron en la categoría "Videojuegos" pero nunca en "Hogar".

```sql
SELECT DISTINCT users.name AS nombre
FROM users
JOIN orders        ON users.id = orders.user_id
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
JOIN categories    ON products.category_id = categories.id
WHERE categories.name = 'Videojuegos'
AND users.id NOT IN (
    SELECT DISTINCT u2.id FROM users u2
    JOIN orders o2        ON u2.id = o2.user_id
    JOIN order_product op2 ON o2.id = op2.order_id
    JOIN products p2       ON op2.product_id = p2.id
    JOIN categories c2     ON p2.category_id = c2.id
    WHERE c2.name = 'Hogar'
)
```

---

## Consulta 27 — Top 3 ciudades por ingresos
**`GET /api/top-ciudades`**

Ranking de las 3 ciudades con mayores ingresos totales.

```sql
SELECT users.city                    AS ciudad,
       ROUND(SUM(orders.total), 2)   AS ingresos_totales
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.city
ORDER BY ingresos_totales DESC
LIMIT 3
```

---

## Consulta 28 — Orden con mayor variedad de productos
**`GET /api/orden-mayor-variedad`**

Devuelve la orden que incluye la mayor cantidad de productos distintos.

```sql
SELECT orders.order_number                              AS numero_orden,
       COUNT(DISTINCT order_product.product_id)        AS productos_distintos
FROM orders
JOIN order_product ON orders.id = order_product.order_id
GROUP BY orders.id, orders.order_number
ORDER BY productos_distintos DESC
LIMIT 1
```

---

## Consulta 29 — Productos vendidos a precio menor
**`GET /api/productos-vendidos-precio-menor`**

Identifica productos que fueron vendidos a un precio inferior al precio actual en catálogo.

```sql
SELECT DISTINCT products.name   AS producto,
                products.price  AS precio_actual,
                order_product.price AS precio_vendido
FROM products
JOIN order_product ON order_product.product_id = products.id
WHERE order_product.price < products.price
ORDER BY products.name
```

---

## Consulta 30 — Historial de un producto
**`GET /api/historial-producto/:nombre`**

Muestra quién compró un producto específico, cuándo y a qué precio.

```sql
SELECT users.name               AS comprador,
       orders.created_at        AS fecha_compra,
       order_product.price      AS precio_pagado,
       order_product.quantity   AS cantidad
FROM products
JOIN order_product ON order_product.product_id = products.id
JOIN orders        ON order_product.order_id = orders.id
JOIN users         ON orders.user_id = users.id
WHERE products.name = ?
ORDER BY orders.created_at DESC
```

---

## Consulta 31 — Usuarios sobre el promedio de gasto
**`GET /api/usuarios-sobre-promedio`**

Lista los usuarios cuyo gasto total supera el promedio global usando subconsulta con `HAVING`.

```sql
SELECT users.name                    AS nombre,
       ROUND(SUM(orders.total), 2)   AS gasto_total
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name
HAVING gasto_total > (
    SELECT AVG(gasto_por_usuario) FROM (
        SELECT SUM(total) AS gasto_por_usuario
        FROM orders GROUP BY user_id
    ) AS sub
)
ORDER BY gasto_total DESC
```

---

## Consulta 32 — Productos estrella
**`GET /api/productos-estrella`**

Productos que representan más del 2% del ingreso total de la tienda.

```sql
SELECT products.name                                                            AS producto,
       ROUND(SUM(order_product.price * order_product.quantity), 2)             AS ingreso_producto,
       ROUND(SUM(order_product.price * order_product.quantity) * 100.0 /
         (SELECT SUM(price * quantity) FROM order_product), 2)                 AS porcentaje
FROM products
JOIN order_product ON order_product.product_id = products.id
GROUP BY products.id, products.name
HAVING porcentaje > 2
ORDER BY porcentaje DESC
```

---

## Consulta 33 — Churn rate (usuarios inactivos)
**`GET /api/churn-rate`**

Detecta usuarios que no han realizado pedidos en los últimos 6 meses.

```sql
SELECT users.name              AS nombre,
       MAX(orders.created_at)  AS ultimo_pedido
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name
HAVING ultimo_pedido < NOW() - INTERVAL 6 MONTH
ORDER BY ultimo_pedido ASC
```

---

## Consulta 34 — Clasificación de clientes
**`GET /api/clasificacion-clientes`**

Clasifica a los clientes en **VIP** (>$5,000), **Frecuente** (≥$1,000) o **Regular** usando `CASE WHEN`.

```sql
SELECT users.name                    AS nombre,
       ROUND(SUM(orders.total), 2)   AS gasto_total,
       CASE
           WHEN SUM(orders.total) > 5000  THEN 'VIP'
           WHEN SUM(orders.total) >= 1000 THEN 'Frecuente'
           ELSE 'Regular'
       END                           AS clasificacion
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name
ORDER BY gasto_total DESC
```

---

## Consulta 35 — Mejor mes de facturación
**`GET /api/mejor-mes`**

Devuelve el mes con la mayor facturación total registrada.

```sql
SELECT YEAR(created_at)       AS anio,
       MONTH(created_at)      AS mes,
       MONTHNAME(created_at)  AS nombre_mes,
       ROUND(SUM(total), 2)   AS facturacion
FROM orders
GROUP BY YEAR(created_at), MONTH(created_at), MONTHNAME(created_at)
ORDER BY facturacion DESC
LIMIT 1
```

---

## Consulta 36 — Alerta de inventario
**`GET /api/alerta-inventario`**

Muestra las órdenes pendientes que incluyen productos con stock menor a 5 unidades.

```sql
SELECT orders.order_number  AS orden,
       products.name        AS producto,
       products.stock       AS stock_actual
FROM orders
JOIN order_product ON orders.id = order_product.order_id
JOIN products      ON order_product.product_id = products.id
WHERE orders.status = 'pending' AND products.stock < 5
ORDER BY products.stock ASC
```

---

## Consulta 37 — Porcentaje de ventas por categoría
**`GET /api/porcentaje-ventas-categoria`**

Calcula el ingreso y porcentaje de participación de cada categoría sobre el total de ventas.

```sql
SELECT categories.name                                                          AS categoria,
       ROUND(SUM(order_product.price * order_product.quantity), 2)             AS ingreso,
       ROUND(SUM(order_product.price * order_product.quantity) * 100.0 /
         (SELECT SUM(price * quantity) FROM order_product), 2)                 AS porcentaje
FROM categories
JOIN products      ON products.category_id = categories.id
JOIN order_product ON order_product.product_id = products.id
GROUP BY categories.id, categories.name
ORDER BY porcentaje DESC
```

---

## Consulta 38 — Ventas por ciudad vs. promedio global
**`GET /api/ventas-ciudad-vs-promedio`**

Compara las ventas de cada ciudad contra el promedio global e indica si está por encima o por debajo.

```sql
SELECT users.city                   AS ciudad,
       ROUND(SUM(orders.total), 2)  AS ventas_ciudad,
       ROUND((SELECT AVG(total_ciudad) FROM
           (SELECT SUM(o.total) AS total_ciudad FROM orders o
            JOIN users u ON o.user_id = u.id GROUP BY u.city) AS p), 2) AS promedio_global,
       CASE WHEN SUM(orders.total) > (SELECT AVG(t) FROM
           (SELECT SUM(o2.total) AS t FROM orders o2
            JOIN users u2 ON o2.user_id = u2.id GROUP BY u2.city) x)
       THEN 'Por encima' ELSE 'Por debajo' END                          AS comparacion
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.city
ORDER BY ventas_ciudad DESC
```

---

## Consulta 39 — Tasa de cancelación mensual
**`GET /api/tasa-cancelacion`**

Muestra por mes el total de órdenes, cuántas fueron canceladas y la tasa porcentual.

```sql
SELECT YEAR(created_at)                                                         AS anio,
       MONTHNAME(created_at)                                                    AS mes,
       COUNT(*)                                                                 AS total_ordenes,
       COUNT(CASE WHEN status = 'cancelled' THEN 1 END)                        AS canceladas,
       ROUND(COUNT(CASE WHEN status='cancelled' THEN 1 END)*100.0/COUNT(*),2)  AS tasa_pct
FROM orders
GROUP BY YEAR(created_at), MONTH(created_at), MONTHNAME(created_at)
ORDER BY anio, MONTH(created_at)
```

---

## Consulta 40 — Pares de productos frecuentes
**`GET /api/pares-productos`**

Encuentra los 10 pares de productos que se compran juntos con más frecuencia (market basket analysis).

```sql
SELECT p1.name      AS producto_1,
       p2.name      AS producto_2,
       COUNT(*)     AS veces_juntos
FROM order_product a
JOIN order_product b ON a.order_id = b.order_id
                    AND a.product_id < b.product_id
JOIN products p1    ON a.product_id = p1.id
JOIN products p2    ON b.product_id = p2.id
GROUP BY a.product_id, b.product_id, p1.name, p2.name
ORDER BY veces_juntos DESC
LIMIT 10
```

---
