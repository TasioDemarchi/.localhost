---
tags: [dev/practices, dev/arquitectura]
tema: Decisiones estructurales del proyecto
proyecto: my-finance-app
---

# Buenas Prácticas — Arquitectura

Decisiones estructurales que afectan cómo se organiza y se comunica la aplicación completa.

---

## Arquitectura en capas (Layered Architecture)

**Aplica a:** Backend — cualquier aplicación con lógica de negocio + persistencia
**Decisión:** Separar en tres capas estrictas: Controller → Service → Repository
**Por qué:** Cada capa tiene una sola responsabilidad. Se puede testear, reemplazar o extender sin tocar las demás.
**En el proyecto:** `LoanController` no hace queries. `LoanRepository` no calcula vencimientos. `LoanService` no sabe de HTTP.
**Ver más:** [[Capas de una Aplicacion Web]]

---

## Single Responsibility Principle (SRP)

**Aplica a:** Todo el codebase
**Decisión:** Cada clase hace una sola cosa. Si una clase necesita dos razones para cambiar, se divide.
**Por qué:** Las clases con múltiples responsabilidades son imposibles de testear y de entender.
**En el proyecto:** `IcbcStatementParser` solo parsea PDFs de ICBC. No guarda en DB. No valida reglas de negocio.

---

## Open/Closed Principle (OCP)

**Aplica a:** Módulos que van a crecer con nuevas variantes
**Decisión:** Definir interfaces desde el inicio para que agregar variantes no requiera modificar código existente.
**Por qué:** Modificar código que funciona introduce riesgos. Agregar código nuevo es más seguro.
**En el proyecto:** `StatementParser` es una interfaz. Agregar soporte para Galicia = nueva clase. El resto no cambia.
**Ver más:** [[ADR - Architecture Decision Records]]

---

## Contrato primero, implementación después

**Aplica a:** Módulos con múltiples implementaciones posibles
**Decisión:** Diseñar la interfaz (el contrato) antes de escribir la implementación concreta.
**Por qué:** Obliga a pensar en qué necesita el consumidor, no en cómo se implementa.
**En el proyecto:** `StatementParser` fue definida antes de escribir `IcbcStatementParser`.

---

## Monorepo para proyectos personales o de equipo pequeño

**Aplica a:** Proyectos con múltiples módulos
**Decisión:** Un solo repositorio con carpetas por módulo.
**Por qué:** Historial unificado, cambios atómicos, un solo entorno de desarrollo.
**Ver más:** [[Monorepo]]

---

## Documentar decisiones, no solo código (ADR)

**Aplica a:** Cualquier proyecto que dure más de un sprint
**Decisión:** Cada decisión técnica importante queda en un ADR con contexto, decisión, consecuencias y alternativas.
**Por qué:** El código dice QUÉ. El ADR dice POR QUÉ. Sin el POR QUÉ, el conocimiento muere con quien decidió.
**Ver más:** [[ADR - Architecture Decision Records]]

---

## Separación web / mobile mediante API compartida

**Aplica a:** Proyectos con frontend web y app mobile
**Decisión:** Backend + web primero. El mobile consume la misma API REST.
**Por qué:** Evita duplicar lógica de negocio. La API es el contrato compartido entre todos los clientes.
**En el proyecto:** El backend no sabe si el cliente es React o React Native. Devuelve JSON.
