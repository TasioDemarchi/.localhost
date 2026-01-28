
# Novedades Esenciales de Java Moderno (De Java 8 a Java 21)

Este documento resume las características más utilizadas en el desarrollo moderno con Spring Boot, enfocándose en la reducción de código repetitivo (boilerplate) y la programación declarativa.

---

## 1. Mejoras de Sintaxis y Legibilidad

### Inferencia de Tipos con var (Java 10)

Permite al compilador deducir el tipo de la variable local basándose en el valor asignado.

- Objetivo: Reducir la verbosidad en declaraciones largas.
    
- Regla: Usar solo cuando el nombre de la variable hace obvio su contenido.
    

``` java
// Antes
Map<String, List<Usuario>> usuarios = new HashMap<>();

// Ahora
var usuarios = new HashMap<String, List<Usuario>>();
```

### Bloques de Texto / Text Blocks (Java 15)

Cadenas de texto multilínea que preservan el formato. Ideal para SQL, JSON o HTML embebido.

- Objetivo: Eliminar la concatenación (+) y los caracteres de escape excesivos.

```java
// Antes
String json = "{\n" + " \"nombre\": \"Dev\"\n" + "}";

// Ahora
String json = """

              {

                "nombre": "Dev"

              }

              """;
```

### Switch Expressions (Java 14 -> 21)

El switch evolucionado que puede retornar valores directamente y usa sintaxis de flecha (->) para evitar el error de olvidar el break.

```Java
// Ahora (Como expresión funcional)
int dias = switch (mes) {
    case "ENERO", "MARZO" -> 31;
    case "ABRIL" -> 30;
    default -> throw new IllegalArgumentException("Mes inválido");
};
```

Para devolver un valor de un bloque de código se utiliza `yield`
```java

```
## 2. Modelado de Datos

### Records (Java 16)

Una forma concisa de definir clases inmutables que solo portan datos (DTOs).

- Objetivo: Eliminar el boilerplate de POJOs (constructores, getters, equals, hashCode, toString).
    
- Uso: Ideal para DTOs en respuestas REST o configuraciones.
    

```java
// Antes (POJO clásico de 50 líneas)
public class UsuarioDTO { ... }

// Ahora (En una sola línea)
public record UsuarioDTO(String nombre, String email) {}
```
  

### Pattern Matching para instanceof (Java 16)

Combina la comprobación de tipo y el casting (conversión) en una sola instrucción.

```java
// Antes
if (obj instanceof String) {
    String s = (String) obj; // Cast manual redundante
    System.out.println(s.length());
}

// Ahora
if (obj instanceof String s) { // Variable 's' creada automáticamente

    System.out.println(s.length());
}
```

## 3. Programación Funcional y Manejo de Nulos

### Optional `<T>`(La "Caja")

Un contenedor que puede o no contener un valor no nulo.

- Objetivo: Evitar NullPointerException y forzar al programador a decidir qué hacer si el valor falta.
    
- Filosofía: No trabajar con el objeto directo, sino con la caja.
    
```java
// Ejemplo: Buscar usuario y obtener su nombre o un valor por defecto
String nombre = repositorio.findById(1) // Devuelve Optional<Usuario>
        .map(Usuario::getNombre)        // Si existe, extrae nombre
        .orElse("ANÓNIMO");             // Si estaba vacío, devuelve esto
```
- Métodos clave: 
	- .orElse() 
	- .orElseThrow() 
	- .map()
	- .isPresent(): Devuelve true si el valor existe, false si no existe
	- .get(): Retorna el valor, si no existe retorna `NoSuchElementException`

### Stream API (La "Cinta Transportadora")

Una abstracción para procesar secuencias de elementos de forma declarativa (qué quiero) en lugar de imperativa (cómo recorrerlo).

- Objetivo: Reemplazar bucles for complejos con operaciones encadenables.
    
- Flujo: Fuente (Lista) -> Operaciones Intermedias (Transformación) -> Operación Terminal (Resultado).
    
```java
// Ejemplo: Obtener emails de usuarios activos
List<String> emails = usuarios.stream()
    .filter(Usuario::isActivo)    // 1. Filtrar (Intermedia)
    .map(Usuario::getEmail)       // 2. Transformar a String (Intermedia)
    .toList();                    // 3. Empaquetar en lista (Terminal)
```

## 4. Bonus: Concurrencia (Avanzado)

### Virtual Threads (Java 21)

Hilos ligeros gestionados por la JVM en lugar del sistema operativo. Permiten crear millones de hilos para aplicaciones de alto rendimiento sin cambiar el modelo de programación. Ideal para aumentar el throughput en aplicaciones I/O intensivas (como APIs REST).

  