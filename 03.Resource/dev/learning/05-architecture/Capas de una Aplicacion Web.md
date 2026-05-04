---
tags: [dev/learning, dev/arquitectura, dev/backend, dev/spring]
tema: Arquitectura de aplicaciones web
proyecto: my-finance-app
---

# Capas de una Aplicación Web

## El problema sin capas

Imaginá que escribís todo en un solo archivo: la conexión a la base de datos, la lógica de negocio, la validación del input y la respuesta HTTP. Al principio funciona. Pero cuando querés:

- **Testear** solo la lógica de negocio → no podés, está mezclada con la DB
- **Cambiar PostgreSQL** por otro motor → tenés que tocar todo el archivo
- **Reutilizar** la lógica de "calcular cuota vencida" en otro lugar → la copiás y pegás

Esto tiene nombre: **código espagueti**. Las capas existen para evitarlo.

---

## Las capas del backend

```
┌─────────────────────────────────────┐
│            Controller               │  ← Recibe requests HTTP, devuelve responses
├─────────────────────────────────────┤
│             Service                 │  ← Lógica de negocio pura
├─────────────────────────────────────┤
│           Repository                │  ← Acceso a la base de datos
├─────────────────────────────────────┤
│            Database                 │  ← PostgreSQL
└─────────────────────────────────────┘
```

> [!important] Regla de oro
> **Cada capa habla solo con la capa inmediatamente debajo.** El Controller llama al Service. El Service llama al Repository. El Controller NUNCA llama al Repository directamente.

---

## Controller — La puerta de entrada

Recibe el request HTTP y lo delega. **No contiene lógica de negocio.**

```java
@RestController
@RequestMapping("/api/loans")
public class LoanController {

    private final LoanService loanService;

    @GetMapping("/{id}")
    public ResponseEntity<LoanResponse> getLoan(@PathVariable UUID id) {
        return ResponseEntity.ok(loanService.findById(id)); // delega al service
    }
}
```

El Controller solo sabe de **HTTP**: códigos de estado (200, 404, 400), headers, request body. No sabe de SQL ni de reglas de negocio.

---

## Service — El cerebro

Contiene la lógica de negocio. **No sabe nada de HTTP ni de SQL.**

```java
@Service
public class LoanService {

    private final LoanRepository loanRepository;

    public LoanResponse findById(UUID id) {
        Loan loan = loanRepository.findById(id)
            .orElseThrow(() -> new LoanNotFoundException(id));

        // lógica de negocio: calcular cuántas cuotas quedan, próximo vencimiento, etc.
        return LoanMapper.toResponse(loan);
    }
}
```

> [!tip] Test del Service
> Si podés testear el Service **sin levantar una base de datos** (usando un mock del Repository), la separación está bien hecha.

---

## Repository — El traductor a SQL

Solo sabe hablar con la base de datos. **No contiene lógica de negocio.**

```java
@Repository
public interface LoanRepository extends JpaRepository<Loan, UUID> {

    List<Loan> findByUserIdAndStatus(UUID userId, LoanStatus status);

    @Query("SELECT l FROM Loan l WHERE l.nextDueDate <= :date")
    List<Loan> findLoansWithDueDateBefore(LocalDate date);
}
```

Spring Data JPA genera la implementación automáticamente. En la mayoría de los casos no necesitás escribir SQL — el nombre del método es suficiente.

---

## Por qué esta separación importa

| Escenario | Sin capas | Con capas |
|-----------|-----------|-----------|
| Testear lógica de negocio sin la DB | ❌ imposible | ✅ mockeo el Repository |
| Cambiar PostgreSQL por otro motor | ❌ toco todo | ✅ solo toco el Repository |
| Exponer la misma lógica por HTTP y por un cron job | ❌ duplico código | ✅ el Service no sabe quién lo llama |
| Agregar caché a las queries | ❌ mezclado con lógica | ✅ agrego `@Cacheable` en el Repository |

---

## El flujo de un request real

Cuando la app mobile pide "mostrar el préstamo #123":

```
📱 Mobile
   ↓ GET /api/loans/123
🌐 Controller  → valida UUID → llama LoanService.findById(id)
⚙️  Service    → busca en repo → calcula cuotas pendientes → mapea a DTO
🗄️  Repository → SELECT * FROM loans WHERE id = '...'
💾 Database   → devuelve la fila
   ↑ sube por las capas
📱 Mobile recibe: { id, monto, cuotasTotal, cuotasPendientes, proximoVencimiento }
```

---

## DTO — Data Transfer Object

El Service devuelve un `LoanResponse`, no un `Loan`. ¿Por qué?

La entidad `Loan` tiene **todos los campos de la base de datos**, incluyendo cosas que el cliente no necesita (IDs internos, `created_at`, relaciones lazy). El DTO contiene **exactamente lo que el cliente necesita** — ni más, ni menos.

```
Loan (entidad JPA)        LoanResponse (DTO)
──────────────────        ──────────────────────────
id                    →   id
userId                    (omitido — el cliente ya sabe quién es)
amount                →   amount
currencyId            →   currency  (el código: "ARS", "USD")
createdAt                 (omitido)
updatedAt                 (omitido)
installments          →   installmentsTotal
                      →   installmentsPaid     (calculado en el Service)
                      →   nextDueDate          (calculado en el Service)
```

> [!warning] Nunca exponer entidades JPA directamente
> Si exponés la entidad, acoplás tu contrato de API a tu modelo interno de datos. Cualquier cambio en la entidad (renombrar un campo, agregar una relación) rompe la API.

---

## Estructura de carpetas

```
backend/src/main/java/com/myfinance/
├── controller/
│   └── LoanController.java
├── service/
│   └── LoanService.java
├── repository/
│   └── LoanRepository.java
├── domain/
│   └── Loan.java              ← entidad JPA
├── dto/
│   ├── LoanRequest.java       ← lo que entra
│   └── LoanResponse.java      ← lo que sale (Record)
└── exception/
    └── LoanNotFoundException.java
```

---

## Relación con otros conceptos

- [[ERD - Diagrama Entidad-Relacion]] → Las entidades del ERD se convierten en clases `@Entity` en el dominio
- [[Migraciones - Flyway]] → Flyway crea las tablas que JPA mapea
- [[ADR - Architecture Decision Records]] → ADR-004 aplica este patrón para `StatementParser`

---

## Preguntas de validación

> [!question] Pregunta 1
> Tenés que enviar un email cuando una cuota está a 3 días de vencer. ¿En qué capa va esa lógica — Controller, Service o Repository? ¿Por qué?

*(Respondé antes de avanzar)*

> [!question] Pregunta 2
> ¿Por qué el Controller no debería hacer un `SELECT` directo a la base de datos? ¿Qué problema concreto crearía eso?

*(Respondé antes de avanzar)*

> [!question] Pregunta 3
> Tenés un Service que calcula el rendimiento de una inversión. El mismo cálculo lo necesitás en un job automático que corre todas las noches. ¿Cómo las capas te facilitan esto?

*(Respondé antes de avanzar)*

> [!question] Pregunta 4
> ¿Qué es un DTO y por qué no devolvemos la entidad JPA directamente al cliente?

*(Respondé antes de avanzar)*
