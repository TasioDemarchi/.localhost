---
tags: [dev/learning, dev/devops, dev/docker, dev/infraestructura]
tema: Entorno de desarrollo reproducible
proyecto: my-finance-app
---

# Docker — Qué es y por qué lo usamos

## El problema que resuelve

Instalaste PostgreSQL en tu máquina para desarrollar. Todo funciona. Después querés deployar en un servidor. El servidor tiene otra versión de PostgreSQL, otra configuración, otro sistema operativo. Nada funciona igual.

O peor: otro dev quiere correr el proyecto y tiene que seguir 10 pasos para instalar todo. Se olvida uno y nada funciona.

> [!tip] La promesa de Docker
> **"Si funciona en mi máquina, funciona en cualquier máquina."**

---

## ¿Qué es un contenedor?

Un contenedor es un **proceso aislado** que incluye todo lo que necesita para correr: el sistema operativo mínimo, las dependencias, la configuración. Es como una caja sellada y reproducible.

```
Sin Docker:                     Con Docker:
─────────────────               ────────────────────────────────
Tu máquina                      Tu máquina
├── Java 8 (viejo)              ├── Docker Engine
├── PostgreSQL 12               │   ├── contenedor: postgres:16-alpine
├── Node 16                     │   ├── contenedor: eclipse-temurin:21
└── conflictos entre versiones  │   └── contenedor: node:20-alpine
                                └── todo aislado, sin conflictos
```

---

## Docker vs Docker Compose

| Herramienta | Para qué |
|-------------|----------|
| `Docker` | Maneja **un** contenedor individual |
| `Docker Compose` | Orquesta **múltiples** contenedores juntos |

Tu app necesita al menos dos cosas corriendo a la vez: el backend (Java) y la base de datos (PostgreSQL). Docker Compose los define juntos y los levanta **con un solo comando**.

---

## El `docker-compose.yml` del proyecto

```yaml
services:
  postgres:
    image: postgres:16-alpine           # imagen oficial de PostgreSQL 16
    container_name: my-finance-db
    environment:
      POSTGRES_DB: myfinance            # nombre de la base de datos
      POSTGRES_USER: myfinance          # usuario
      POSTGRES_PASSWORD: myfinance_dev  # contraseña (solo dev, no producción)
    ports:
      - "5432:5432"                     # host:contenedor
    volumes:
      - postgres_data:/var/lib/postgresql/data  # persistencia de datos
```

### El campo `ports: "5432:5432"`

Formato: `"puerto_del_host:puerto_del_contenedor"`

Significa: cuando algo en tu máquina se conecta al puerto `5432`, Docker redirige al puerto `5432` del contenedor. Tu app Spring Boot se conecta a `localhost:5432` como si PostgreSQL estuviera instalado localmente.

### El campo `volumes`

Sin volumes, cada vez que parás el contenedor **perdés todos los datos**. Los volumes mapean una carpeta del contenedor a un espacio persistente de tu máquina.

```bash
docker compose down     # para el contenedor — datos se mantienen ✅
docker compose down -v  # para Y borra los volumes — datos perdidos ❌
```

### `alpine` en la imagen

`postgres:16-alpine` es la versión mínima basada en Alpine Linux — un sistema operativo ultra liviano.
- Alpine: ~50MB
- Versión completa: ~200MB

Para desarrollo está perfecto. En producción también se usa habitualmente.

---

## El backend durante desarrollo vs producción

```yaml
# docker-compose.yml — en desarrollo
# El backend lo corrés desde el IDE, no desde Docker
# Descomentar cuando el backend esté listo para containerizar:

# backend:
#   build: ./backend
#   ports:
#     - "8080:8080"
```

**¿Por qué no containerizamos el backend durante el desarrollo?**

Porque cada vez que cambiás código tendrías que reconstruir la imagen Docker. Desde el IDE (IntelliJ), Spring Boot tiene **hot reload** — los cambios se reflejan casi instantáneamente.

En producción sí va en contenedor — ahí no hay IDE y la reproducibilidad importa.

---

## Comandos del día a día

```bash
# Levantar todo (en background)
docker compose up -d

# Ver qué contenedores están corriendo
docker compose ps

# Ver los logs de PostgreSQL
docker compose logs postgres

# Parar todo (los datos se mantienen)
docker compose down

# Parar y BORRAR todos los datos
docker compose down -v

# Reconstruir imágenes (cuando cambiás el Dockerfile)
docker compose up -d --build
```

---

## Seguridad: la contraseña en texto plano

El `docker-compose.yml` tiene la contraseña en texto plano (`myfinance_dev`). **¿Es esto un problema?**

| Contexto | ¿Problema? | Por qué |
|----------|-----------|---------|
| Desarrollo local | ❌ No | Es una contraseña de dev sin datos reales. El archivo está en `.gitignore` o se usa una contraseña genérica |
| Producción | ✅ Sí | Nunca pongas credenciales reales en el compose. Usás variables de entorno o secrets del CI/CD |

> [!warning] Regla del proyecto
> `docker-compose.yml` solo tiene credenciales de desarrollo. Las credenciales de producción van en variables de entorno del servidor — nunca en el repositorio.

---

## Relación con otros conceptos

- [[Monorepo]] → El `docker-compose.yml` vive en la raíz del monorepo para levantar todo el entorno
- [[Migraciones - Flyway]] → Flyway se conecta a PostgreSQL corriendo en Docker y aplica las migraciones al arrancar Spring Boot
- [[Git Workflow - Ramas y PRs]] → Las credenciales de producción nunca llegan al historial de Git

---

## Preguntas de validación

> [!question] Pregunta 1
> ¿Qué significa que un contenedor es "aislado"? ¿En qué se diferencia de instalar PostgreSQL directamente en tu máquina?

*(Respondé antes de avanzar)*

> [!question] Pregunta 2
> Si hacés `docker compose down -v`, ¿qué pasa con los datos que tenías en la base de datos? ¿Cuándo usarías este comando y cuándo NO lo usarías?

*(Respondé antes de avanzar)*

> [!question] Pregunta 3
> El `docker-compose.yml` tiene la contraseña de la base de datos en texto plano. ¿Es un problema? ¿En qué contexto sí y en qué contexto no?

*(Respondé antes de avanzar)*

> [!question] Pregunta 4
> ¿Por qué el backend va a estar en un contenedor en producción, pero durante desarrollo lo corremos directo desde el IDE?

*(Respondé antes de avanzar)*
