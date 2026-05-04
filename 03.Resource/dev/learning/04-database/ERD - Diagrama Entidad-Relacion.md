---
tags: [dev/learning, dev/database, dev/modelado]
tema: Modelado de base de datos
proyecto: my-finance-app
---

# ERD — Diagrama Entidad-Relación

## ¿Qué es?

Un ERD (Entity-Relationship Diagram) es el **mapa de tu base de datos**. Muestra:

- Qué **entidades** existen (tablas)
- Qué **atributos** tiene cada una (columnas)
- Cómo se **relacionan** entre sí (claves foráneas)

> [!important] Analogía
> Es el equivalente al **plano de un edificio**. Antes de poner un ladrillo, el arquitecto dibuja el plano. Antes de escribir una sola línea de SQL, diseñás el ERD.

---

## ¿Por qué diseñarlo antes de codear?

Si arrancás a crear tablas sin el ERD, vas a descubrir a mitad del camino que:

- Guardaste el monto de una cuota sin la moneda → no podés mostrar multi-moneda
- Relacionaste gastos directamente con el usuario en vez de con la tarjeta → no podés filtrar por tarjeta
- No modelaste el tipo de cambio → no podés hacer conversiones históricas

**Cambiar la estructura de una base de datos con datos reales adentro es costoso y arriesgado.** Los errores de modelado que se descubren tarde son los más caros de corregir.

---

## Los tipos de relaciones

### Uno a Muchos (1:N) — el más común

Un usuario tiene muchas tarjetas. Una tarjeta pertenece a un solo usuario.

```
users (1) ──────────── (N) credit_cards
```

En la base de datos, esto se implementa con una **clave foránea en la tabla del lado "muchos"**:

```sql
CREATE TABLE credit_cards (
  id      UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id),  -- FK en el lado "muchos"
  ...
);
```

### Uno a Uno (1:1) — menos común

Un usuario tiene un perfil. Un perfil pertenece a un usuario. Nuestro proyecto no tiene relaciones 1:1 en la Fase 0.

### Muchos a Muchos (N:M) — requiere tabla intermedia

Una categoría puede tener muchos gastos y un gasto puede tener muchas categorías. Esto requiere una tabla intermedia: `transaction_categories(transaction_id, category_id)`.

En nuestro proyecto lo simplificamos: un gasto tiene **una sola categoría** opcional.

---

## Las relaciones del proyecto

```
users           (1) ──── (N) loans
loans           (1) ──── (N) loan_installments

users           (1) ──── (N) credit_cards
credit_cards    (1) ──── (N) card_statements
card_statements (1) ──── (N) card_transactions

users           (1) ──── (N) investments
investment_types(1) ──── (N) investments

currencies      (1) ──── (N) exchange_rates
currencies      (1) ──── (N) investments
currencies      (1) ──── (N) card_transactions
```

---

## Clave primaria vs Clave foránea

| Concepto | Qué es | Ejemplo en el proyecto |
|----------|--------|----------------------|
| **Primary Key (PK)** | Identificador único de cada fila | `users.id` |
| **Foreign Key (FK)** | Referencia a la PK de otra tabla | `credit_cards.user_id` |

---

## ON DELETE: CASCADE vs RESTRICT

La FK define qué pasa si borrás un registro "padre". Hay dos comportamientos principales:

| Comportamiento | Qué hace | Cuándo usarlo |
|---------------|----------|--------------|
| `ON DELETE CASCADE` | Borra en cascada los hijos | Cuando los hijos no tienen sentido sin el padre |
| `ON DELETE RESTRICT` | No te deja borrar si tiene hijos (falla explícito) | **Default en este proyecto** — preferimos que falle a perder datos |

> [!warning] Default del proyecto: RESTRICT
> Usamos `RESTRICT` en todo. Si queremos borrar un `user`, primero tenemos que borrar sus tarjetas, préstamos e inversiones. Es más seguro que `CASCADE`, que puede borrar datos relacionados silenciosamente.

---

## UUID vs Autoincremental

Usamos `UUID` como clave primaria, **no** `SERIAL` (1, 2, 3...).

| Tipo | Características |
|------|----------------|
| `SERIAL` (1, 2, 3...) | Predecible. Un atacante puede adivinar que el recurso `/loans/5` existe |
| `UUID` (a3f8c2d1-...) | Imposible de adivinar. Generado de forma aleatoria |

**Ventajas adicionales del UUID:**
- Podés generar el ID **en el cliente antes de insertar** (útil para mobile offline)
- Al sincronizar datos entre dev y producción, los IDs **no colisionan**
- Compatible con **Hibernate batch inserts** (a diferencia de `IDENTITY`)

> [!note] UUID v4 vs v7
> UUID v7 es monotónico (mejor para índices B-tree) pero Hibernate 6.x no tiene soporte nativo aún. Usamos v4 por ahora.

---

## Las 14 entidades del proyecto

| Entidad | Bounded Context | Descripción |
|---------|----------------|-------------|
| `users` | TRANSVERSAL | Usuario de la app |
| `currencies` | TRANSVERSAL | Catálogo de monedas (ARS, USD, BTC) |
| `exchange_rates` | TRANSVERSAL | Historial de tipos de cambio |
| `categories` | TRANSVERSAL | Árbol de categorías (self-referencial) |
| `notifications` | TRANSVERSAL | Notificaciones genéricas |
| `loans` | LOANS | Préstamos |
| `loan_installments` | LOANS | Cuotas de cada préstamo |
| `credit_cards` | CARDS | Tarjetas de crédito |
| `card_statements` | CARDS | Resúmenes de tarjeta |
| `card_transactions` | CARDS | Transacciones (cada cuota = una fila) |
| `statement_imports` | CARDS | Registro de PDFs importados |
| `investment_types` | INVESTMENTS | Tipos (FCI, acción, cripto, plazo fijo) |
| `investments` | INVESTMENTS | Inversiones registradas |
| `asset_prices` | INVESTMENTS | Historial de precios de activos |

---

## Preguntas de validación

> [!question] Pregunta 1
> Mirá esta relación: `card_statements (1) ──── (N) card_transactions`. En palabras simples, ¿qué significa? ¿Podés dar un ejemplo concreto con datos reales?

*(Respondé antes de avanzar)*

> [!question] Pregunta 2
> ¿Por qué `loan_installments` tiene una FK hacia `loans` y no al revés? ¿Qué pasaría si lo modelaras al revés?

*(Respondé antes de avanzar)*

> [!question] Pregunta 3
> Si borrás una `credit_card`, ¿qué debería pasar con sus `card_statements` y `card_transactions`? ¿Usarías `CASCADE` o `RESTRICT`? ¿Por qué?

*(Respondé antes de avanzar)*

> [!question] Pregunta 4
> ¿Por qué la tabla `currencies` existe como tabla separada y no como un `VARCHAR` directamente en cada tabla con montos? ¿Qué ventaja da?

*(Respondé antes de avanzar)*
