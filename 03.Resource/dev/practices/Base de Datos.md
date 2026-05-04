---
tags: [dev/practices, dev/database, dev/postgresql]
tema: Modelado, tipos, migraciones y naming en PostgreSQL
proyecto: my-finance-app
---

# Buenas Prácticas — Base de Datos

---

## BigDecimal para montos monetarios (nunca Double)

**Aplica a:** Cualquier campo que represente dinero
**Decisión:** `NUMERIC(18,4)` en PostgreSQL → `BigDecimal` en Java. Nunca `FLOAT`, `DOUBLE` o `DECIMAL` con poca precisión.
**Por qué:** Los tipos de punto flotante tienen errores de representación binaria. `0.1 + 0.2 = 0.30000000000000004` en Double. Con dinero eso es inaceptable.
**En el proyecto:** Todos los campos `amount`, `exchange_rate`, `price` usan `NUMERIC(18,4)` y `BigDecimal`.

---

## UUID como clave primaria

**Aplica a:** Todas las tablas
**Decisión:** `UUID` generado con `gen_random_uuid()` como PK, no `SERIAL`.
**Por qué:** Los IDs autoincrementales son predecibles. UUID es imposible de adivinar y permite generar el ID en el cliente antes de insertar (útil para mobile offline). También es compatible con Hibernate batch inserts.
**Ver más:** [[ERD - Diagrama Entidad-Relacion]]

---

## Snapshot del tipo de cambio por transacción

**Aplica a:** Sistemas multi-moneda
**Decisión:** Cada transacción guarda el tipo de cambio vigente al momento del registro, además de la referencia a la moneda.
**Por qué:** El tipo de cambio fluctúa. Si reconstruís el valor histórico usando el TC actual, obtenés un número incorrecto.
**En el proyecto:** `card_transactions.exchange_rate_snapshot`, `investments.exchange_rate_at_entry`.

---

## Tabla de catálogo para monedas (no VARCHAR libre)

**Aplica a:** Valores que son un conjunto cerrado y conocido
**Decisión:** La moneda es una FK a la tabla `currencies`, no un `VARCHAR` libre.
**Por qué:** Un `VARCHAR` libre permite inconsistencias ("usd", "USD", "Usd"). La FK garantiza integridad referencial y permite agregar metadata.
**En el proyecto:** `currencies(id, code, name, symbol)` — todas las entidades con monto referencian esta tabla.

---

## Migraciones versionadas con Flyway (nunca DDL manual)

**Aplica a:** Cualquier cambio a la estructura de la base de datos
**Decisión:** Todo cambio de esquema es un archivo SQL versionado en `db/migration/`. Nunca DDL manual en producción.
**Por qué:** La base de datos tiene que poder reconstruirse desde cero en cualquier entorno. Sin esto, los entornos se desincronizán silenciosamente.
**Ver más:** [[Migraciones - Flyway]]

---

## Inmutabilidad de migraciones aplicadas

**Aplica a:** Archivos Flyway ya ejecutados
**Decisión:** Nunca editar un archivo de migración ya ejecutado. Si hay un error, se crea una migración nueva.
**Por qué:** Flyway guarda el checksum de cada migración. Si cambiás el archivo, falla con error en todos los entornos donde ya fue aplicado.
**Ver más:** [[Migraciones - Flyway]]

---

## Datos de referencia en migraciones (seed)

**Aplica a:** Datos que deben existir en todos los entornos
**Decisión:** Los datos de catálogo constantes del dominio van en una migración de seed, no en código de aplicación.
**Por qué:** Son parte del esquema funcional. Que estén en Flyway garantiza que existen antes de que la app arranque.
**En el proyecto:** `V3__seed_currencies.sql` inserta ARS, USD, BTC, ETH.

---

## ON DELETE RESTRICT como default

**Aplica a:** Todas las relaciones con FK
**Decisión:** `ON DELETE RESTRICT` por defecto. `CASCADE` solo cuando se documenta explícitamente por qué.
**Por qué:** `RESTRICT` falla explícito si intentás borrar un padre con hijos. Es más seguro que `CASCADE`, que puede borrar datos relacionados silenciosamente.

---

## Naming en snake_case para tablas y columnas

**Aplica a:** Toda la base de datos
**Decisión:** Nombres en `snake_case` minúscula: `loan_installments`, `exchange_rate_snapshot`, `created_at`.
**Por qué:** PostgreSQL es case-insensitive por default. `snake_case` es la convención universal en SQL y evita necesitar comillas para nombres con mayúsculas.
