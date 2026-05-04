---
tags: [dev/moc, dev/indice]
tema: Índice general de desarrollo
proyecto: my-finance-app
---

# Dev — Mapa de Contenido

Índice central de todos los conceptos y prácticas del proyecto **my-finance-app**.

---

## 📚 Learning — Conceptos explicados en profundidad

### 01 · Fundamentos del proyecto
- [[Monorepo]] — Qué es, por qué lo usamos, cuándo no usarlo
- [[Conventional Commits]] — Formato, tipos, scopes, por qué importa el historial
- [[ADR - Architecture Decision Records]] — Qué son, cuándo usarlos, los 4 ADRs del proyecto

### 04 · Base de datos
- [[ERD - Diagrama Entidad-Relacion]] — Tipos de relaciones, las 14 entidades del proyecto, UUID vs SERIAL
- [[Migraciones - Flyway]] — Formato de archivos, cómo funciona, la regla crítica de inmutabilidad

### 05 · Arquitectura
- [[Capas de una Aplicacion Web]] — Controller / Service / Repository, DTOs, flujo de un request

### 06 · DevOps
- [[Git Workflow - Ramas y PRs]] — main → develop → feature, ciclo de vida de una feature
- [[Docker - Introduccion]] — Contenedores, Docker Compose, volumes, desarrollo vs producción

---

## ✅ Practices — Decisiones concretas y replicables

- [[Arquitectura]] — Capas, SRP, OCP, monorepo, ADRs, API compartida
- [[Backend]] — Java 21, Records, LAZY, EnumType.STRING, REST naming, DTOs, tests
- [[Base de Datos]] — BigDecimal, UUID, snapshot de TC, FK catalog, Flyway, RESTRICT, snake_case
- [[Frontend]] — TypeScript strict, Vite, TanStack Query, shadcn/ui, container/presentacional, RHF
- [[Git y Workflow]] — Conventional commits, ramas, PRs, commits atómicos, gitignore, secretos
- [[Diseno y Producto]] — Dominio primero, datos caros, MVP, funcionar antes que bonito, scope por fase

---

## 🗺️ Estado del proyecto

| Fase | Descripción | Estado |
|------|-------------|--------|
| Fase 0 | Dominio, ERD, diseño de entidades JPA | ✅ Completa |
| Fase 1 | Backend Spring Boot + API REST + migraciones | 🔄 Próxima |
| Fase 2 | Frontend React + TypeScript | 🔲 Pendiente |
| Fase 3 | Integraciones (PDF parser, tipo de cambio API) | 🔲 Pendiente |
| Fase 4 | Mobile React Native | 🔲 Pendiente |
