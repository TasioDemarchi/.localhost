---
tags: [dev/practices, dev/producto, dev/ux]
tema: Decisiones sobre alcance, producto y diseño
proyecto: my-finance-app
---

# Buenas Prácticas — Diseño y Producto

---

## Diseñar el dominio antes de tocar código

**Aplica a:** Inicio de cualquier proyecto o módulo nuevo
**Decisión:** Antes de escribir una clase o una tabla, modelar el dominio en papel o en un documento. Glosario, entidades y relaciones primero.
**Por qué:** El costo de cambiar un modelo de datos con datos reales adentro es alto. El costo de cambiar un documento es cero.
**En el proyecto:** El ERD y el glosario de dominio existieron antes del primer archivo Java.
**Ver más:** [[ERD - Diagrama Entidad-Relacion]]

---

## Las decisiones de datos son las más caras de cambiar

**Aplica a:** Cualquier decisión que afecte el esquema de base de datos
**Decisión:** Anticipar los casos de uso futuros en el modelo de datos, aunque la feature no se implemente aún.
**Por qué:** Agregar una columna nueva es barato. Cambiar el tipo de una columna existente con millones de filas es una operación de riesgo.
**En el proyecto:** Multi-moneda se integró desde el día cero aunque en V1 solo se use ARS y USD.

---

## MVP primero — web antes que mobile

**Aplica a:** Orden de construcción de las partes del sistema
**Decisión:** Backend + web primero. Mobile es un cliente más que consume la misma API.
**Por qué:** Construir dos frontends a la vez con pocas horas por semana no es viable. Primero se valida el dominio y la API.
**En el proyecto:** Fases 1 y 2 (backend + web) antes de la Fase 4 (mobile).

---

## Primero hace funcionar, después hace bonito

**Aplica a:** Orden de prioridades en cada feature
**Decisión:** La lógica de negocio correcta primero. El diseño visual y las optimizaciones después.
**Por qué:** Es más fácil hacer bonito algo que funciona bien que hacer funcionar algo que se ve bien.
**En el proyecto:** Los endpoints de la API se testean con Swagger antes de que exista cualquier interfaz gráfica.

---

## Abstraer desde el primer caso, no después

**Aplica a:** Módulos con variantes conocidas desde el diseño
**Decisión:** Si se sabe que habrá más de una implementación, definir la abstracción con el primer caso.
**Por qué:** Refactorizar código con datos en producción para agregar una abstracción es costoso y arriesgado. Si la variante futura es conocida, el costo de la abstracción inicial es mínimo.
**En el proyecto:** `StatementParser` como interfaz desde la primera implementación de ICBC, sabiendo que vendrán Galicia, BBVA, etc.
**Ver más:** [[Arquitectura]]

---

## Alcance explícito por fase

**Aplica a:** Planificación del proyecto
**Decisión:** Cada fase tiene un alcance fijo y acordado. Las features que no entran van al backlog — no se agregan al vuelo.
**Por qué:** El scope creep (agregar cosas sobre la marcha) es la causa más común de proyectos que nunca terminan.
**En el proyecto:** La Fase 1 termina cuando la API está documentada, testeada y con migraciones. Punto.
