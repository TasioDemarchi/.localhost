---
tags: [dev/practices, dev/backend, dev/java, dev/spring]
tema: Java 21, Spring Boot 3, API REST
proyecto: my-finance-app
---

# Buenas Prácticas — Backend

Decisiones de Java 21, Spring Boot 3, API REST, testing y manejo de errores.

---

## Sin Lombok — Java 21 es expresivo por diseño

**Aplica a:** Todo el código Java del backend
**Decisión:** No usar Lombok. Usar Records de Java 21 para value objects e inmutables. Getters explícitos donde se necesiten.
**Por qué:** Lombok oculta código detrás de anotaciones mágicas. Cuando algo falla, el stack trace es confuso. Java 21 tiene Records que resuelven lo mismo de forma nativa y transparente.
**En el proyecto:** `ParsedTransaction`, `ParsedStatement` son Records. Las entidades JPA tienen getters explícitos.

---

## Records para value objects, entidades para estado mutable

**Aplica a:** Diseño de clases Java
**Decisión:** `record` para objetos inmutables entre capas (DTOs, resultados de parseo). `@Entity` solo para clases que se persisten en DB.
**Por qué:** Los Records son inmutables por diseño — no tienen setters. Las entidades JPA necesitan ser mutables para que Hibernate las gestione.
**En el proyecto:** `LoanResponse` es un Record. `Loan` es una `@Entity`.

---

## LAZY fetch por defecto en todas las relaciones JPA

**Aplica a:** Todas las anotaciones `@OneToMany`, `@ManyToOne`, `@ManyToMany`
**Decisión:** `fetch = FetchType.LAZY` en todas las relaciones. Se carga con `JOIN FETCH` o `@EntityGraph` solo donde se justifica.
**Por qué:** EAGER carga datos que quizás no se usan, generando queries innecesarias. Con LAZY, solo se trae lo que se pide explícitamente.
**En el proyecto:** Un `Loan` no carga sus `LoanInstallment` automáticamente.

---

## @Enumerated(EnumType.STRING) — nunca ORDINAL

**Aplica a:** Todos los enums persistidos en base de datos
**Decisión:** `@Enumerated(EnumType.STRING)` siempre.
**Por qué:** `ORDINAL` guarda el número de posición del enum. Si reordenás o insertás un valor en el medio, todos los registros existentes quedan con datos incorrectos.
**En el proyecto:** `LoanStatus.ACTIVE` se guarda como `"ACTIVE"`, no como `0`.

---

## API REST — Naming de endpoints en plural y sustantivos

**Aplica a:** Diseño de la API REST
**Decisión:** Recursos en plural (`/loans`, `/investments`). Nunca verbos en la URL.
**Por qué:** REST es un estilo basado en recursos, no en acciones. El verbo HTTP (GET, POST, PUT, DELETE) ya indica la acción.
**Ejemplo:**
```
GET    /api/loans       → listar
GET    /api/loans/{id}  → obtener uno
POST   /api/loans       → crear
PATCH  /api/loans/{id}  → actualizar parcialmente
DELETE /api/loans/{id}  → borrar
```

---

## DTOs para todo lo que entra y sale de la API

**Aplica a:** Controllers — request body y response body
**Decisión:** Nunca exponer entidades JPA directamente. Usar DTOs de entrada (Request) y de salida (Response).
**Por qué:** Las entidades tienen campos internos que no deberían exponerse. Exponer la entidad acopla el contrato de la API al modelo interno de datos.
**En el proyecto:** `LoanRequest` para crear. `LoanResponse` para devolver. `Loan` nunca sale del Service.
**Ver más:** [[Capas de una Aplicacion Web]]

---

## Manejo centralizado de errores

**Aplica a:** Toda la API REST
**Decisión:** Un solo `@RestControllerAdvice` maneja todas las excepciones y las convierte a respuestas HTTP con formato consistente.
**Por qué:** Sin esto, cada Controller tiene su propio try/catch y los errores llegan al cliente en formatos distintos.
**En el proyecto:** `GlobalExceptionHandler` — un lugar para mapear `LoanNotFoundException` → 404, `ValidationException` → 400.

---

## Cobertura mínima de tests: 70% antes de mergear

**Aplica a:** Todo código que va a `develop`
**Decisión:** El Service layer debe tener al menos 70% de cobertura con tests unitarios. Los Repositories con tests de integración usando Testcontainers.
**Por qué:** Los tests son la red de seguridad que permite refactorizar y agregar features sin romper lo que ya funciona.
**En el proyecto:** Tests en `src/test/java/`. Testcontainers levanta PostgreSQL real en los tests de integración.
