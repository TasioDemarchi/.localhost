---
tags: [dev/learning, dev/git, dev/workflow]
tema: Gestión de código y historial
proyecto: my-finance-app
---

# Conventional Commits — Por qué el historial de Git importa

## El problema

Mirá estos dos historiales de Git del mismo proyecto:

**Historial A — sin convención:**
```
fix stuff
cambios
update
arreglé el bug ese
más cambios frontend
wip
```

**Historial B — con Conventional Commits:**
```
feat(backend): agregar endpoint de registro de cuota de préstamo
fix(frontend): corregir cálculo de rendimiento en vista de inversiones
chore(db): agregar migración V3 para tabla asset_prices
docs(adr): documentar decisión de multi-moneda
refactor(backend): extraer lógica de conversión de moneda a servicio propio
```

> [!tip] La diferencia clave
> El Historial B te dice **qué cambió, dónde, y qué tipo de cambio fue** — sin abrir ningún archivo ni leer ningún diff.

El historial de Git es **documentación**. Si no lo tratás como tal, perdés contexto valioso cada vez que commitiás.

---

## El formato

```
<tipo>(<scope>): <descripción en infinitivo>
```

Ejemplo real del proyecto:
```
feat(backend): agregar endpoint de registro de cuota
  ↑      ↑              ↑
tipo   scope        descripción
```

---

## Los tipos disponibles

| Tipo | Cuándo usarlo | Ejemplo |
|------|--------------|---------|
| `feat` | Funcionalidad nueva | `feat(backend): agregar login con JWT` |
| `fix` | Corrección de bug | `fix(frontend): corregir fecha de vencimiento` |
| `refactor` | Cambio interno **sin** cambio de comportamiento | `refactor(backend): simplificar parser de PDF` |
| `test` | Agrega o modifica tests | `test(backend): cubrir casos borde en conversión de moneda` |
| `docs` | Solo documentación | `docs: agregar ADR de tipo de cambio` |
| `chore` | Mantenimiento, configuración | `chore: actualizar dependencias de Spring Boot` |
| `style` | Formato, espacios, sin cambio de lógica | `style(frontend): aplicar prettier en módulo de inversiones` |
| `perf` | Mejora de rendimiento | `perf(backend): agregar índice en card_transactions.statement_id` |

---

## Los scopes del proyecto

```
backend   → código Java / Spring Boot
frontend  → código React / TypeScript
mobile    → código React Native
db        → migraciones Flyway
docs      → documentación y ADRs
ci        → GitHub Actions y pipelines
```

> [!note] Cuándo omitir el scope
> Cuando el cambio afecta a todo el monorepo y no tiene un módulo específico. Por ejemplo: `chore: initial monorepo setup`.

---

## Por qué el infinitivo

"agregar", no "agregué" ni "agrega".

La convención internacional dice que el mensaje describe **qué hace el commit si lo aplicás**, no qué hiciste vos. Es como la descripción de un parche: "este parche agrega validación", no "este parche agregó validación".

```
✅ feat(backend): agregar validación de monto negativo
❌ feat(backend): agregué validación de monto negativo
❌ feat(backend): agrega validación de monto negativo
```

---

## El primer commit del proyecto

```
chore: initial monorepo setup with project structure and ADRs
```

- `chore` porque es configuración inicial, no una feature
- Sin scope porque afecta todo el monorepo
- En inglés — es válido, lo que importa es ser consistente

---

## Analogía

Pensá en los commits como entradas de un diario de obra. Si el capataz escribe "hice cosas" cada día, en 3 meses nadie sabe qué se construyó cuándo. Si escribe "se colocaron vigas en el piso 3, sector norte", en 3 meses podés rastrear exactamente qué pasó y cuándo.

---

## Preguntas de validación

> [!question] Pregunta 1
> Escribís el módulo de inversiones desde cero. ¿Cómo sería el mensaje de commit cuando terminás y funciona?

> [!success] Tu respuesta
> `feat(backend): agregar módulo de gestión de inversiones`
> 
> ✅ Correcto en esencia. Nota: si terminaste el módulo completo de una vez, podés no usar scope porque afecta backend + posiblemente db. Ahí `feat: agregar módulo de inversiones` también es válido.

> [!question] Pregunta 2
> Encontrás un bug: cuando importás el PDF de ICBC, las cuotas al contado se importan con `installment_number = 0` en vez de `null`. ¿Cómo sería el commit?

> [!success] Tu respuesta
> `fix(backend): corregir valor de installment_number para cuotas al contado en parser ICBC`
> 
> ✅ Perfecto. Específico, localizado, en infinitivo.

> [!question] Pregunta 3
> ¿Por qué existe `refactor` separado de `feat`? ¿Qué diferencia hace para alguien que lee el historial?

> [!success] Tu respuesta
> `feat` agrega algo nuevo o modifica un comportamiento visible. `refactor` cambia el código internamente (estructura, legibilidad, extracción de lógica) sin cambiar lo que hace la aplicación desde afuera. Para quien lee el historial, un `refactor` garantiza que **no se cambió el comportamiento** — puede ignorarlo si solo le importa qué features hay.
