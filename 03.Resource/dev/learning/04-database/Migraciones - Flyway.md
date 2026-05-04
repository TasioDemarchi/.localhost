---
tags: [dev/learning, dev/database, dev/flyway, dev/migraciones]
tema: Gestión de esquema de base de datos
proyecto: my-finance-app
---

# Migraciones de Base de Datos — Flyway

## El problema

La base de datos **cambia con el tiempo**. Hoy tenés la tabla `loans`. Mañana necesitás agregar la columna `notes` en `loan_installments`. Pasado necesitás una tabla nueva `asset_prices`.

¿Cómo sabés qué versión de la base de datos está corriendo en cada entorno (tu máquina, el servidor)? ¿Cómo sincronizás esos cambios sin perder datos?

> [!danger] Sin una herramienta de migraciones
> La respuesta es: *"manualmente, con miedo"*. Entornos que se desincronizán silenciosamente. "En mi máquina funciona" pero en producción falla porque alguien olvidó correr un SQL.

---

## ¿Qué es Flyway?

Flyway es una herramienta que administra la evolución de tu base de datos a través de **archivos SQL versionados**. Cada cambio a la base de datos es un archivo. Flyway lleva registro de cuáles ya se aplicaron.

---

## El formato de los archivos

```
V1__create_users.sql
V2__create_currencies.sql
V3__seed_currencies.sql
V4__create_loans.sql
V5__add_notes_to_loan_installments.sql
```

Formato: `V{número}__{descripción}.sql`

| Parte | Regla |
|-------|-------|
| `V` | Mayúscula siempre |
| `{número}` | Orden de ejecución — nunca se repite |
| `__` | Dos guiones bajos (separador obligatorio) |
| `{descripción}` | Guiones bajos en vez de espacios |

---

## Cómo funciona internamente

Flyway mantiene una tabla interna llamada `flyway_schema_history`:

```
flyway_schema_history
─────────────────────────────────────────────────────────
version | description          | success | checksum
─────────────────────────────────────────────────────────
1       | create users         | true    | 1234567
2       | create currencies    | true    | 2345678
3       | seed currencies      | true    | 3456789
```

**Cuando arranca tu app, Flyway:**
1. Mira qué archivos hay en `src/main/resources/db/migration/`
2. Compara con lo que ya está en `flyway_schema_history`
3. Ejecuta **solo las que faltan**, en orden numérico

```
Tu máquina:      V1 ✅  V2 ✅  V3 ✅  V4 ✅  V5 ✅
Servidor prod:   V1 ✅  V2 ✅  V3 ✅
                                          ↑ al deployar, aplica V4 y V5 automáticamente
```

---

## La regla más importante

> [!danger] NUNCA modifiques un archivo de migración que ya fue ejecutado
> Flyway guarda el **checksum** (huella digital) de cada archivo. Si lo modificás después de ejecutarlo, falla con:
> ```
> ERROR: Migration checksum mismatch for migration version 3
> ```
> 
> **Si cometiste un error en V3:** la solución es crear V4 que lo corrija. Nunca editar V3.

---

## Ejemplos concretos del proyecto

```sql
-- V1__create_users.sql
CREATE TABLE users (
    id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email      VARCHAR(255) NOT NULL UNIQUE,
    name       VARCHAR(255) NOT NULL,
    created_at TIMESTAMP    NOT NULL DEFAULT NOW()
);

-- V2__create_currencies.sql
CREATE TABLE currencies (
    id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code   VARCHAR(10)  NOT NULL UNIQUE,  -- ARS, USD, BTC
    name   VARCHAR(100) NOT NULL,
    symbol VARCHAR(10)  NOT NULL          -- $, U$S, ₿
);

-- V3__seed_currencies.sql  ← datos de catálogo, siempre necesarios
INSERT INTO currencies (code, name, symbol) VALUES
    ('ARS', 'Peso Argentino',       '$'  ),
    ('USD', 'Dólar Estadounidense', 'U$S'),
    ('BTC', 'Bitcoin',              '₿'  ),
    ('ETH', 'Ethereum',             'Ξ'  );
```

---

## Flyway en Spring Boot

Con Spring Boot 3, Flyway se integra **automáticamente**:

1. Agregás la dependencia en `pom.xml`
2. Ponés los archivos SQL en `src/main/resources/db/migration/`
3. Spring los ejecuta al arrancar la app — **no hay que hacer nada más**

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

> [!tip] Cero código para orquestar
> No hay que llamar a Flyway manualmente ni escribir código de inicialización. Arrancás el servidor y la base de datos se actualiza sola.

---

## Migraciones de datos (seed) vs migraciones de estructura

Flyway no distingue entre las dos — para él son solo SQL. La convención del proyecto es:

| Tipo | Ejemplo | Cuándo usarlo |
|------|---------|--------------|
| Estructura (DDL) | `CREATE TABLE`, `ALTER TABLE`, `ADD COLUMN` | Cambios al esquema |
| Datos (seed) | `INSERT INTO currencies` | Datos de catálogo que deben existir en **todos** los entornos |

Los datos de seed van en Flyway porque son parte del esquema funcional — si no existen, la app no puede operar correctamente.

---

## Relación con otros conceptos

- [[ERD - Diagrama Entidad-Relacion]] → El ERD define qué tablas crear; Flyway las crea con SQL
- [[Docker - Introduccion]] → PostgreSQL corre en Docker; Flyway se conecta a él al arrancar

---

## Preguntas de validación

> [!question] Pregunta 1
> Creaste la tabla `loans` en V4. Después te diste cuenta que te olvidaste de agregar la columna `interest_rate`. ¿Qué hacés: editás V4 o creás V5? ¿Por qué?

*(Respondé antes de avanzar)*

> [!question] Pregunta 2
> ¿Por qué es valioso que Flyway ejecute las migraciones automáticamente al arrancar la app? ¿Qué problema concreto evita?

*(Respondé antes de avanzar)*

> [!question] Pregunta 3
> Tenés V1 a V8 aplicadas en producción. En tu máquina estás en V10 (dos migraciones nuevas). ¿Qué pasa exactamente cuando hacés deploy a producción?

*(Respondé antes de avanzar)*

> [!question] Pregunta 4
> ¿Por qué la migración `V3__seed_currencies.sql` que inserta datos tiene sentido como migración de Flyway, si Flyway "es para estructura de base de datos"?

*(Respondé antes de avanzar)*
