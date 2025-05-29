# Documentación sobre Triggers en SQL Server

Este documento explora en detalle el concepto de **Triggers** (disparadores) en **Microsoft SQL Server**, sus usos, ventajas, desventajas y buenas prácticas.

---

## Tabla de Contenidos

* [¿Qué es un Trigger?](#1-qué-es-un-trigger)
* [¿Para qué sirve un Trigger?](#2-para-qué-sirve-un-trigger)
* [¿Cuándo se puede usar un Trigger?](#3-cuándo-se-puede-usar-un-trigger)
* [Estructura de un Trigger en SQL Server](#4-estructura-de-un-trigger-en-sql-server)
* [Tipos de Trigger en SQL Server](#5-tipos-de-trigger-en-sql-server)
* [¿Cuándo ejecuta su acción un Trigger?](#6-cuándo-ejecuta-su-acción-un-trigger)
* [Ejemplo de un Trigger](#7-ejemplo-de-un-trigger)
* [Trigger Marketing (concepto externo)](#8-trigger-marketing)
* [¿Cómo hacer un Trigger?](#9-cómo-hacer-un-trigger)
* [Características de un Trigger](#10-características-de-un-trigger)
* [Desventajas de un Trigger](#11-desventajas-de-un-trigger)
* [Alternativas a los Triggers](#12-alternativas-a-los-triggers)

---

## 1. ¿Qué es un Trigger?

Un **Trigger** (o disparador) en SQL Server es un objeto de base de datos que ejecuta automáticamente una serie de instrucciones en respuesta a un evento como un `INSERT`, `UPDATE` o `DELETE` sobre una tabla o vista.

Se define en el servidor y actúa como un "listener" que responde a cambios en los datos.

**Ventajas principales:**

* Automatización de tareas.
* Validación y consistencia de datos.
* Auditoría de cambios.

---

## 2. ¿Para qué sirve un Trigger?

Los triggers en SQL Server se utilizan para:

* Auditoría de cambios (quién, cuándo, qué cambió).
* Validación de reglas de negocio complejas.
* Control de integridad adicional.
* Automatización de actualizaciones o acciones secundarias.
* Mantenimiento de datos agregados o sincronizados.

---

## 3. ¿Cuándo se puede usar un Trigger?

Se recomienda su uso cuando:

* Se requiere lógica consistente y automática tras una modificación.
* No se puede confiar en que la aplicación controle ciertos cambios.
* Se necesita trazabilidad de operaciones directamente en la base.
* Se gestionan operaciones sobre vistas actualizables mediante `INSTEAD OF`.

---

## 4. Estructura de un Trigger en SQL Server

### Sintaxis general

```sql
CREATE TRIGGER nombre_trigger
ON nombre_tabla
[AFTER | INSTEAD OF] [INSERT, UPDATE, DELETE]
[WITH <opciones>] -- Opcional
[NOT FOR REPLICATION] -- Opcional
AS
BEGIN
  SET NOCOUNT ON;

  -- Lógica del trigger
END;
```

---

### Componentes principales (SQL Server)

* `AFTER`: Se ejecuta después de la operación.
* `INSTEAD OF`: Se ejecuta en lugar de la operación (especialmente útil en vistas).
* **Pseudo-tablas**:

  * `INSERTED`: contiene los nuevos datos (`INSERT` o `UPDATE`).
  * `DELETED`: contiene los datos anteriores (`DELETE` o `UPDATE`).

SQL Server no utiliza `BEFORE` triggers ni `FOR EACH ROW`. Los triggers son de tipo "statement-level" por defecto y deben trabajar con conjuntos de datos si múltiples filas son afectadas.

---

### Ejemplo con pseudo-tablas

```sql
UPDATE products
SET products.last_updated = CURRENT_TIMESTAMP
FROM products 
INNER JOIN INSERTED i ON products.id = i.id;
```

Este ejemplo actualiza la columna `last_updated` cuando un producto es modificado.

---

## 5. Tipos de Trigger en SQL Server

### Por tipo de ejecución:

* `AFTER`: Se ejecuta después de la operación (`INSERT`, `UPDATE`, `DELETE`).
* `INSTEAD OF`: Se ejecuta en lugar de la operación.

SQL Server no permite triggers `BEFORE` como otros motores (por ejemplo, MySQL).

---

## 6. ¿Cuándo ejecuta su acción un Trigger?

| Tipo         | Momento de ejecución | Uso común                                |
| ------------ | -------------------- | ---------------------------------------- |
| `AFTER`      | Después del cambio   | Auditoría, validaciones finales          |
| `INSTEAD OF` | En lugar del cambio  | Vistas no actualizables, lógica especial |

---

## 7. Ejemplo de un Trigger

### Tabla de ejemplo: `products`

```sql
CREATE TABLE products (
  id INT IDENTITY PRIMARY KEY,
  product_name VARCHAR(100),
  last_updated DATETIME
);
```

### Vista: 

![Image](https://github.com/user-attachments/assets/d24c1b51-3006-4496-a781-a8832c81e5ae)

### Objetivo

Actualizar automáticamente la columna `last_updated` cuando se modifica un producto.

### Código del Trigger

```sql
CREATE TRIGGER ts_products_update_timestamp 
ON products
AFTER UPDATE
AS
BEGIN 
  SET NOCOUNT ON;

  UPDATE products
  SET products.last_updated = CURRENT_TIMESTAMP
  FROM products p
  INNER JOIN INSERTED i ON products.id = i.id;
END;
```

### Prueba

```sql
UPDATE products
SET product_name = 'Laptop de Juegos'
WHERE id = 1;
```

### Vista Actual:

![Image](https://github.com/user-attachments/assets/f13ac862-6f2c-4adc-b5b1-13f041efb02f)

---

## 8. Trigger Marketing (concepto externo)

**Trigger marketing** no está relacionado con SQL Server o bases de datos. Se refiere a campañas de marketing automatizadas que se ejecutan en respuesta a eventos específicos del usuario.

Ejemplos:

* Envío de correo tras el registro.
* Promociones por cumpleaños.
* Recuperación tras inactividad.
* Recordatorio tras abandono de carrito.

Esto se configura en plataformas como HubSpot, ActiveCampaign o Mailchimp.

---

## 9. ¿Cómo hacer un Trigger?

Pasos para crear un trigger en SQL Server:

- Define el evento: `INSERT`, `UPDATE` o `DELETE`.
- Elige el tipo: `AFTER` o `INSTEAD OF`.
- Asocia la tabla o vista.
- Escribe la lógica usando `INSERTED` y/o `DELETED`.
- Incluye `SET NOCOUNT ON;` para evitar mensajes innecesarios.
- Asegúrate de que la lógica funcione con múltiples filas.

---

## 10. Características de un Trigger

* Reacción automática a cambios.
* Ejecutado dentro de la transacción que dispara el evento.
* Transparente para el usuario y la aplicación.
* Uso de pseudo-tablas (`INSERTED`, `DELETED`).
* Puede modificar otras tablas.
* Permite lógica condicional compleja.

---

## 11. Desventajas de un Trigger

* Difíciles de depurar (no se ejecutan directamente).
* Lógica no visible desde la aplicación.
* Puede afectar el rendimiento si no se controla bien.
* Posibles problemas de concurrencia.
* Dependencia de SQL Server (poca portabilidad).
* Riesgo de bucles o recursividad mal gestionada.
* Aumenta la complejidad del mantenimiento.

---

## 12. Alternativas a los Triggers

En algunos contextos, es mejor evitar triggers y optar por:

* Lógica de negocio en la aplicación (Java, C#, Python, etc.).
* Procedimientos almacenados llamados directamente.
* Arquitecturas orientadas a eventos (Kafka, RabbitMQ).
* Change Data Capture (CDC) en SQL Server.
* Hooks o callbacks en frameworks ORM (Entity Framework, Hibernate).
* Procesamiento de flujos en bases NoSQL (por ejemplo, MongoDB Change Streams).

---

## Nota final

Los triggers en SQL Server son herramientas poderosas para manejar lógica reactiva directamente en la base de datos. Sin embargo, deben usarse con moderación, una planificación adecuada y considerando su impacto en el rendimiento, la claridad del sistema y el mantenimiento a largo plazo.

