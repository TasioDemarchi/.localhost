---
tags: [dev/practices, dev/git, dev/workflow]
tema: Gestión del código, historial y flujo de trabajo
proyecto: my-finance-app
---

# Buenas Prácticas — Git y Workflow

---

## Conventional Commits en todo el historial

**Aplica a:** Todos los commits del proyecto
**Decisión:** Formato `<tipo>(<scope>): <descripción en infinitivo>`.
**Por qué:** El historial de Git es documentación. Un commit bien escrito dice qué cambió, dónde y por qué — sin abrir el diff.
**Ver más:** [[Conventional Commits]]

---

## Estrategia de ramas: main → develop → feature

**Aplica a:** Gestión del repositorio
**Decisión:** `main` solo recibe merges desde `develop`. Features y fixes se desarrollan en ramas propias y se mergean a `develop` via Pull Request.
**Por qué:** `main` representa siempre código deployable y estable.
**Ver más:** [[Git Workflow - Ramas y PRs]]

---

## Pull Requests — incluso en proyectos personales

**Aplica a:** Todo merge a `develop` o `main`
**Decisión:** Abrir un PR para cada feature, aunque seas el único desarrollador.
**Por qué:** El PR fuerza una revisión antes de mergear. GitHub guarda el contexto de por qué se hizo cada cosa. Es práctica real de trabajo en equipo.

---

## Commits atómicos — un cambio por commit

**Aplica a:** Todos los commits
**Decisión:** Cada commit hace una sola cosa. Si el mensaje necesita "y", son dos commits.
**Por qué:** Un commit atómico es reversible con `git revert` sin efectos secundarios. Un commit mezcla es imposible de revertir parcialmente.
**En el proyecto:** Agregar la entidad y agregar los tests son dos commits separados.

---

## .gitignore completo desde el día uno

**Aplica a:** Setup inicial del repositorio
**Decisión:** El `.gitignore` cubre desde el inicio: IDE, OS, secretos, build artifacts de cada módulo.
**Por qué:** Una vez que algo sensible llega al historial de Git, eliminarlo completamente es complejo. Prevenir es infinitamente más barato.

---

## Secretos fuera del repositorio — siempre

**Aplica a:** Credenciales, API keys, contraseñas
**Decisión:** Ninguna credencial real va en el repositorio. Variables de entorno en desarrollo, secrets del CI/CD en producción.
**Por qué:** Un repositorio "privado" puede volverse público. Los secretos en el historial de Git son prácticamente imposibles de eliminar de forma segura.
**En el proyecto:** `application-local.yml` está en `.gitignore`. El `docker-compose.yml` usa contraseñas solo para desarrollo local.
