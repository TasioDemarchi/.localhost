---
tags: [dev/learning, dev/arquitectura, dev/proyecto]
tema: Organización de proyectos
proyecto: my-finance-app
---

# Monorepo — Qué es y por qué importa

## El problema que resuelve

Cuando un proyecto tiene múltiples partes (backend, frontend, mobile), tenés dos opciones para organizar el código:

**Opción A — Repos separados:**
```
github.com/vos/my-finance-backend
github.com/vos/my-finance-frontend
github.com/vos/my-finance-mobile
```

**Opción B — Monorepo:**
```
github.com/vos/my-finance-app
├── backend/
├── frontend/
└── mobile/
```

> [!tip] ¿Cuál elegimos?
> **Monorepo**. Para un proyecto personal o de equipo pequeño, las ventajas son claras y los costos de mantenimiento son mínimos.

---

## ¿Por qué monorepo?

| Ventaja | Explicación |
|---------|------------|
| **Un solo historial de Git** | Ves en un lugar todos los cambios del proyecto |
| **Cambios atómicos** | Si cambiás la API y el frontend a la vez, es un solo commit |
| **Más fácil de navegar** | No tenés que cambiar de repositorio para trabajar |
| **Un solo `docker-compose.yml`** | Levantás todo el entorno con un comando |
| **Refactors globales** | Un rename en la API se ve afectado en todos los módulos en el mismo PR |

---

## ¿Cuándo NO usar monorepo?

Cuando el proyecto escala con muchos equipos independientes. Google, Meta y Microsoft usan monorepos gigantes, pero con herramientas específicas como **Bazel**, **Nx** o **Turborepo**. Sin esas herramientas, un monorepo con 50 equipos se vuelve un cuello de botella.

**Regla práctica:** Si el equipo es menor a ~10 personas o el proyecto tiene 2-3 módulos, monorepo es la decisión correcta.

---

## La estructura del proyecto

```
my-finance-app/
├── .gitignore          ← qué archivos NO subir a GitHub
├── README.md           ← descripción del proyecto
├── docker-compose.yml  ← entorno de desarrollo local (levanta todo)
├── backend/            ← API REST (Java 21 + Spring Boot 3)
├── frontend/           ← Web app (React + TypeScript)
├── mobile/             ← Android/iOS app (React Native)
├── docs/
│   ├── CONVENTIONS.md  ← reglas del proyecto
│   ├── adr/            ← decisiones técnicas documentadas
│   └── diagrams/       ← ERD, flujos, diagramas de arquitectura
├── learning/           ← notas de estudio (esta carpeta)
└── practices/          ← buenas prácticas del proyecto
```

> [!important] Principio clave
> Cada carpeta de primer nivel es un **módulo independiente**. El backend no sabe nada del frontend y viceversa. Se comunican **solo a través de la API REST**.

---

## El módulo `docs/adr/`

Los ADR (Architecture Decision Records) viven acá. Son documentos cortos que explican **por qué** se tomó cada decisión técnica importante. Ver → [[ADR - Architecture Decision Records]]

---

## Analogía

Pensalo como un edificio. El monorepo es el edificio entero. Cada piso (backend, frontend, mobile) es independiente — tiene su propia entrada, sus propias reglas. Pero comparten la misma dirección, el mismo portero (Git), y el mismo plano original (el repo).

Los repos separados serían edificios distintos en distintas calles. Cuando querés coordinar algo entre ellos, tenés que caminar de uno al otro.

---

## Preguntas de validación

> [!question] Pregunta 1
> Si cambiás el nombre de un endpoint en el backend, ¿en qué otra carpeta del monorepo tenés que hacer cambios también?

> [!success] Tu respuesta
> En `frontend/` y en `mobile/` — donde se hace el llamado a ese endpoint. Si el nombre (ruta) cambia, todos los clientes que la consumen deben actualizarse.

> [!question] Pregunta 2
> ¿Por qué el `docker-compose.yml` está en la raíz del monorepo y no dentro de `backend/`?

> [!success] Tu respuesta
> Porque el compose levanta **todas las partes del proyecto** con un solo comando. Si estuviera en `backend/`, solo sabría levantar el backend. La raíz es el único lugar que tiene visibilidad de todos los módulos.

> [!question] Pregunta 3
> ¿Para qué sirven los archivos `docs/adr/ADR-001-multimoneda.md`? ¿Quién los lee?

> [!success] Tu respuesta
> Sirven para registrar el contexto y razonamiento detrás de decisiones importantes que afectan al sistema. Los lee cualquier desarrollador (incluyendo vos mismo en 6 meses) que quiera entender *por qué* el código está como está, no solo *qué* hace.
