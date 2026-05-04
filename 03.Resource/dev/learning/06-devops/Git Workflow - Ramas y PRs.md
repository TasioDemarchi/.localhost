---
tags: [dev/learning, dev/git, dev/workflow]
tema: Gestión de ramas y trabajo en equipo
proyecto: my-finance-app
---

# Git Workflow — Cómo trabajamos con ramas

## El problema de trabajar solo en `main`

Si todo el código va directo a `main`, un commit roto puede dejar el proyecto en un estado que no funciona. En producción eso significa que la app cae. En desarrollo significa que no podés avanzar hasta arreglarlo.

Las ramas resuelven esto: **cada feature o fix vive en su propio espacio aislado** hasta que está listo.

---

## El flujo de ramas del proyecto

```
main          ← producción — solo código estable y deployado
  └── develop ← integración — donde se juntan las features terminadas
        ├── feat/loan-module       ← una feature
        ├── feat/credit-card-crud  ← otra feature en paralelo
        └── fix/installment-null   ← un bug fix
```

### `main`
- Solo recibe merges desde `develop` cuando hay una versión lista para deploy
- **Nunca** se commitea directamente aquí
- Cada merge a main = una versión deployada

### `develop`
- Base para todas las features
- Cuando terminás una feature, la mergeas acá
- Es el "trabajo en progreso" estable

### `feat/<nombre>` y `fix/<nombre>`
- Creás una nueva rama para cada feature o fix
- Trabajás ahí sin afectar nada más
- Cuando terminás, abrís un Pull Request hacia `develop`

---

## El ciclo de vida de una feature

```bash
# 1. Siempre partís desde develop actualizado
git checkout develop
git pull origin develop

# 2. Creás tu rama
git checkout -b feat/loan-installments

# 3. Trabajás, hacés commits atómicos
git add .
git commit -m "feat(backend): agregar entidad LoanInstallment con relación a Loan"
git commit -m "test(backend): agregar tests de LoanInstallmentRepository"

# 4. Subís tu rama a GitHub
git push origin feat/loan-installments

# 5. Abrís un Pull Request en GitHub: feat/loan-installments → develop

# 6. Revisás, todo ok, mergeás

# 7. Borrás la rama (ya no la necesitás)
git branch -d feat/loan-installments
```

---

## Pull Request — para qué sirve en un proyecto personal

En equipos, el PR es para que otros revisen tu código. En un proyecto personal **sigue siendo útil** porque:

- Te fuerza a **revisar tus propios cambios** antes de mergear
- GitHub guarda el **historial de por qué** se hizo cada merge
- Podés agregar comentarios explicando decisiones tomadas en esa feature
- Es **práctica real** de trabajo en equipo

> [!tip] Hábito profesional
> Trabajar con PRs aunque seas el único desarrollador es uno de los hábitos que más diferencia a un dev junior de uno senior. Cuando trabajes en un equipo, ya lo vas a tener incorporado.

---

## Traer cambios de develop a tu rama

Si mientras trabajás en tu rama, alguien (o vos mismo desde otra rama) mergea algo a `develop`, podés traerlo:

```bash
# Desde tu rama feat/investments
git merge develop
# o
git rebase develop  # historial más limpio, pero más complejo
```

La opción más segura para empezar: `git merge develop`.

---

## Comandos del día a día

```bash
git status                      # ver qué cambió
git diff                        # ver exactamente qué líneas cambiaron
git log --oneline               # historial compacto
git checkout -b nombre-rama     # crear y moverse a una rama nueva
git checkout nombre-rama        # moverse a una rama existente
git merge nombre-rama           # traer los cambios de una rama a la actual
git pull origin develop         # traer los últimos cambios de develop
git push origin nombre-rama     # subir tu rama a GitHub
git branch -d nombre-rama       # borrar rama local (después de mergear)
```

---

## Relación con otros conceptos

- [[Conventional Commits]] → Los commits dentro de las ramas siguen el formato convencional
- [[ADR - Architecture Decision Records]] → Los ADRs se crean en ramas `docs/` o como parte de la feature que justifican
- [[Monorepo]] → Un solo repositorio, múltiples ramas — todo en el mismo historial

---

## Preguntas de validación

> [!question] Pregunta 1
> Estás trabajando en `feat/investments`. Alguien arregló un bug en la lógica de cuotas y ya lo mergeó a `develop`. ¿Cómo traés ese fix a tu rama actual sin perder tu trabajo?

*(Respondé antes de avanzar)*

> [!question] Pregunta 2
> ¿Por qué nunca deberías hacer `git push origin main` directamente con código sin testear?

*(Respondé antes de avanzar)*

> [!question] Pregunta 3
> Describí con tus palabras el ciclo completo desde que empezás a programar una nueva feature hasta que el código llega a `main`.

*(Respondé antes de avanzar)*
