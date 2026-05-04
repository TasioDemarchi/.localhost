---
tags: [dev/learning, dev/arquitectura, dev/documentacion]
tema: Documentación de decisiones técnicas
proyecto: my-finance-app
---

# ADR — Architecture Decision Records

## ¿Qué es un ADR?

Un ADR es un documento corto que responde la pregunta que todo proyecto eventualmente tiene:

> *"¿Por qué hicimos esto así y no de otra manera?"*

Sin ADRs, esa respuesta vive solo en la cabeza de quien tomó la decisión. Cuando esa persona no está — o cuando pasaron 6 meses — nadie sabe por qué el código está como está.

> [!important] El código dice QUÉ. El ADR dice POR QUÉ.
> Sin el *por qué*, el conocimiento muere con quien tomó la decisión.

---

## ¿Por qué importa en un proyecto personal?

Puede parecer que siendo el único desarrollador no lo necesitás. Pero:

- En **3 meses** vas a leer código tuyo y no recordar por qué lo hiciste así
- Cuando llegues a la **Fase 4 (mobile)**, vas a agradecer tener documentado por qué elegiste React Native y no Kotlin
- Si algún día **alguien más trabaja** en el proyecto, los ADRs son su mapa
- Los ADRs te **obligan a pensar las consecuencias** antes de tomar una decisión — el proceso de escribirlo es tan valioso como el documento en sí

---

## El formato que usamos

```markdown
# ADR-NNN — Título de la decisión

**Fecha:**
**Estado:** proposed | accepted | deprecated | superseded

## Contexto
¿Qué problema fuerza esta decisión?

## Decisión
¿Qué se decide hacer?

## Consecuencias
Positivas y tradeoffs (lo que se gana y lo que se pierde)

## Alternativas consideradas
Qué más se evaluó y por qué se descartó
```

---

## Los estados de un ADR

| Estado | Significado |
|--------|------------|
| `proposed` | Se está evaluando, aún no está confirmada |
| `accepted` | Decisión tomada y activa |
| `deprecated` | Ya no aplica — fue reemplazada o el contexto cambió |
| `superseded` | Fue reemplazada por otro ADR (se referencia cuál) |

---

## Los ADRs del proyecto

### ADR-001 — Multi-moneda desde el día cero
**Decisión:** Toda entidad con monto guarda `amount + currency_id + exchange_rate_snapshot`.  
**Por qué importa:** Agregar multi-moneda después requeriría migrar **toda la base de datos**. Es el tipo de decisión que no podés deshacer barato.

### ADR-002 — Dólar oficial como único tipo de cambio
**Decisión:** Un solo tipo de cambio, actualizado via API.  
**Por qué importa:** En Argentina hay 4+ cotizaciones del dólar (oficial, blue, MEP, CCL). Sin esta decisión, cada pantalla que muestra un monto en USD sería ambigua.

### ADR-003 — Cuotas de tarjeta como filas independientes
**Decisión:** Cada cuota es un registro separado en `card_transactions`.  
**Por qué importa:** Define cómo se importa el PDF y cómo se muestran los datos. Cambiar esto después implica migrar todos los datos de tarjeta existentes.  
**Tradeoff aceptado:** No es posible ver el total original de la compra de forma directa — hay que sumar las cuotas.

### ADR-004 — Parser PDF con interfaz abstracta
**Decisión:** Una interfaz `StatementParser`, una implementación por banco.  
**Por qué importa:** Cuando agregues Galicia o BBVA, no tocás código existente — solo agregás una clase nueva. Esto implementa el **Principio Abierto/Cerrado** (Open/Closed, uno de los SOLID).

---

## ¿Qué NO es un ADR?

- ❌ No es documentación de código (eso va en comentarios o Javadoc)
- ❌ No es un manual de usuario
- ❌ No es una tarea o ticket de trabajo
- ❌ No documenta cada función que escribís — solo decisiones de diseño relevantes

> [!tip] Regla práctica para decidir si algo merece un ADR
> Si la decisión es **reversible y barata** de cambiar → no merece ADR.  
> Si cambiarla después costaría **horas o días** de trabajo → merece ADR.

---

## Relación con otros conceptos

- [[Monorepo]] → Los ADRs viven en `docs/adr/` dentro del monorepo
- [[Conventional Commits]] → Los commits que crean ADRs usan `docs(adr): agregar ADR-004`
- [[Capas de una Aplicación]] → ADR-004 justifica la interfaz `StatementParser`

---

## Preguntas de validación

> [!question] Pregunta 1
> Decidís que los usuarios van a poder tener múltiples perfiles (personal y familiar) en la misma cuenta. ¿Esto merece un ADR? ¿Por qué sí o por qué no?

> [!success] Tu respuesta
> Sí merece un ADR. Es una decisión que cambia la estructura del proyecto (impacta el modelo de datos, las relaciones, la autenticación). Revertirla tiene un costo alto.

> [!question] Pregunta 2
> Mirá el ADR-003. Dice que una consecuencia negativa es "no es posible ver el total de una compra originalmente". ¿Podés pensar en un caso concreto donde eso sea un problema?

> [!success] Tu respuesta
> Si comprás algo en 12 cuotas de $5.000 y querés ver cuánto pagaste en total, tenés que hacer el cálculo mental (12 × $5.000 = $60.000). La app no te muestra el total de la compra directamente — solo las cuotas individuales.

> [!question] Pregunta 3
> ¿Qué significa que el estado de un ADR sea `deprecated`? ¿Cuándo pasaría eso en nuestro proyecto?

> [!success] Tu respuesta
> `deprecated` significa que esa decisión ya no aplica — el contexto cambió o se tomó un camino diferente. En el proyecto podría pasar si, por ejemplo, decidimos agregar soporte para el dólar blue además del oficial. El ADR-002 quedaría `deprecated` y un nuevo ADR-005 lo reemplazaría.
