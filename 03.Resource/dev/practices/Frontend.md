---
tags: [dev/practices, dev/frontend, dev/react, dev/typescript]
tema: React, TypeScript, componentes y estado
proyecto: my-finance-app
---

# Buenas Prácticas — Frontend

> Este archivo crece a medida que avanza la Fase 2 (frontend).

---

## TypeScript estricto — nunca `any`

**Aplica a:** Todo el código TypeScript
**Decisión:** `strict: true` en `tsconfig.json`. Prohibido usar `any`. Si no sabés el tipo, usás `unknown` y lo refinás.
**Por qué:** `any` desactiva el sistema de tipos. Es como apagar el cinturón de seguridad porque molesta. El compilador deja de protegerte.
**En el proyecto:** `tsconfig.json` con `"strict": true` desde el primer commit del frontend.

---

## Vite como bundler — no Create React App

**Aplica a:** Setup del proyecto React
**Decisión:** Usar Vite como bundler y dev server.
**Por qué:** Create React App está deprecado oficialmente. Vite es 10-100x más rápido en desarrollo, tiene HMR instantáneo y configuración más simple.
**En el proyecto:** `frontend/` iniciado con `npm create vite@latest -- --template react-ts`.

---

## TanStack Query para estado del servidor

**Aplica a:** Toda llamada a la API REST
**Decisión:** Usar TanStack Query (React Query) para fetching, caching y sincronización. No usar `useState` + `useEffect` para llamadas a la API.
**Por qué:** `useState` + `useEffect` para fetching es propenso a bugs (race conditions, memory leaks, loading/error inconsistentes). TanStack Query resuelve todo con una API declarativa.
**En el proyecto:** `useLoan(id)`, `useInvestments()` son hooks de TanStack Query.

---

## shadcn/ui para componentes — no MUI ni Ant Design

**Aplica a:** Librería de componentes de UI
**Decisión:** Usar shadcn/ui.
**Por qué:** shadcn no es una dependencia — te da el código fuente de los componentes. Los entendés, los modificás, los hacés tuyos. Con MUI o Ant Design configurás props sin saber qué hay adentro.
**En el proyecto:** Los componentes de UI viven en `frontend/src/components/ui/` como código propio.

---

## Separación container / presentacional

**Aplica a:** Organización de componentes React
**Decisión:** Separar componentes que manejan datos (containers) de componentes que solo renderizan UI (presentacionales).
**Por qué:** Los componentes presentacionales son reutilizables y fáciles de testear porque no tienen dependencias externas.
**En el proyecto:** `LoanListContainer` obtiene los datos. `LoanCard` solo recibe props y renderiza.

---

## React Hook Form para formularios

**Aplica a:** Todos los formularios
**Decisión:** Usar React Hook Form para manejo de formularios y validación.
**Por qué:** Manejar formularios con `useState` por cada campo es verbose y lento (re-render por cada keystroke). React Hook Form usa refs internamente — mínimos re-renders.
**En el proyecto:** Formulario de nueva inversión, registro de gasto manual, import de PDF.
