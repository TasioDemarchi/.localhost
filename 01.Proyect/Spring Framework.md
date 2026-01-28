---
fecha_creacion: 20-12-2025
tags:
  - Java
  - Spring
estado: 🟡 En Progreso
deadline:
---

# Proyecto: Spring Framework

## 🎯 Objetivo
*¿Qué quiero lograr con este proyecto?*
- Entender y poder desarrollar Spring Framework utilizado en proyectos Legacy (Ciudad NTTDATA).
- Entender como navegar por los XML y las configuraciones que se realiza. 
- Aprender nuevas "mecánicas" de Spring que nunca utilice (ej: AOP, Test Junit y Mockito, Manejo de Excepciones, Transacciones, Configuración de entornos).
## Pasos del 1 al 6
1. Aplicaciones empresariales robustas
2. Aprender IoC, DI, Spring Data JPA y Spring Web/MVC
3. Desarrollar un sistema de gestion de finanzas integrando los CRUDs básicos a cada entidad, haciendo uso conciente de los conceptos de IOC y DI, utilizando base de datos SQL y H2 para la persistencia de datos, generando servicios Rest funcionales y aplicando buenas practicas en cada apartado.
### 5. Orden de aprendizaje
- [x] Spring-Core, IoC y DI
	- [x] Configuración de beans (anotaciones y XML)
	- [x] Inyección de dependencias por constructor
	- [x] Crear aplicación de consola donde las clases se comuniquen sin utilizar la palabra `new`
- [x] Arquitectura de capas 
	- [x] Capa `Model`
	- [x] Capa `Repository`
	- [x] Capa `Service`
	- [x] Capa `Controller`
- [ ] Spring Data JPA
	- [ ] Configurar en local base de datos SQL (PostgreSQL, MySQL o SQLServer)
	- [ ] Crear entidades, relaciones, utilizar anotaciones
	- [ ] Consultas personalizadas con Query Method
	- [ ] Consultas con JPQL
	- [ ] Consultas nativas SQL puro
	- [ ] Configurar conexión con base de datos
	- [ ] Validar la persistencia de datos y funciones CRUD sobre los mismos
- [ ] Spring Web/MVC
	- [ ] Concepto e implementación de `DispatcherServlet`
	- [ ] Creación de `Controllers`
	- [ ] Uso correcto de verbos HTTP
	- [ ] Manejo de respuesta con `ResponseEntity<T>`

#### Adicionales
- [ ] Investigar distintas arquitecturas de capas para proyectos Spring
- [ ] DTOs con Java Records
- [ ] Paginación
- [ ] Documentación con OpenAPI/Swagger
- [ ] Comparator
### Recursos
- [Spring Doc](https://spring.io/projects/spring-framework)
- [Baeldung](https://www.baeldung.com/)
- [Pildora Informatica - Curso Spring](https://www.youtube.com/playlist?list=PLU8oAlHdN5Blq85GIxtKjIXdfHPksV_Hm)

## 🧠 Buffer / Notas en Crudo
*Espacio para volcar ideas rápidas, links o dudas mientras trabajo. Procesar y limpiar semanalmente.*

- 
### Temas que veo en el trabajo
- Filtros para cabezeras
- Cors
- Consultas SQL directas en hibernate (Spring 5 y implementación distinta)
# Spring-Core
## IoC (Inversion of Control)
- En Spring el control de la creación de objetos y gestion de dependencias, deja de ser tarea del código y pasa a ser tarea del contenedor de Spring.
## DI (Dependency Inyection)
- Es un patron que implementa Spring que le permite la creación y gestion de dependencias fuera de la clase (evita la palabra `new`), inyectándola en tiempo de ejecución.
	- Inyección por Constructor (Mas prolijo)
	- Inyección por Setter
	- Inyección por Campo (@Autowired)
## Practica
- Spring depende de un archivo de configuración `.xml` o el uso de las anotaciones, habitándolas con `<context:component-scan base-package="${package}$"/>` una clase anotada con `@Configuration` para gestionar los objetos.
- Los objetos definidos en alguno de estas configuraciones pasa a llamarse `beans`, así los nombra Spring, y son los objetos que Spring gestion tanto para su creación como inyección de dependencias.
- En la clase main del proyecto, es necesario iniciar el contexto de Spring por medio de `ClassPathXmlAplicationContext context = new ClassPathXmlAplicationContext(${file});`
- Por medio del método `context.getBean(${beanName});` se obtienen los beans necesarios para el inicio de la aplicación.
### Configuración por XML

```java
public class EmailProvider {
    public void enviar(String mensaje) {
        System.out.println("Enviando email: " + mensaje);
    }
}
---
public class NotificadorServiceXML {
    // Dependencias
    private EmailProvider provider;
    private String prefijoAsunto;
    private int maxReintentos; // Primitivo puro

    // 1. INYECCIÓN POR CONSTRUCTOR
    // Requisito: Definir un constructor con argumentos.
    // Beneficio: Garantiza que el objeto nace "listo para usar". Es inmutable.
    public NotificadorServiceXML(EmailProvider provider, String prefijoAsunto) {
        this.provider = provider;
        this.prefijoAsunto = prefijoAsunto;
    }

    // 2. INYECCIÓN POR SETTER
    // Requisito: Constructor vacío (o por defecto) y métodos Setters públicos.
    // Beneficio: Permite dependencias opcionales o cambiarlas después de crear el objeto.
    public void setMaxReintentos(int maxReintentos) {
        this.maxReintentos = maxReintentos;
    }
    
    public void ejecutar() {
        provider.enviar(prefijoAsunto + " Hola Mundo. Reintentos: " + maxReintentos);
    }
}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="miEmailProvider" class="com.ejemplo.EmailProvider" />

    <bean id="notificadorService" class="com.ejemplo.NotificadorServiceXML">
        
        <constructor-arg index="0" ref="miEmailProvider" />
        
        <constructor-arg index="1" value="[IMPORTANTE]" />

        <property name="maxReintentos" value="5" />
        
    </bean>

</beans>
```
### Configuración en clase
```java
package com.ejemplo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Component;

// @Component permite a Spring detectar esta clase automáticamente
@Component 
public class EmailProvider {
    public void enviar(String mensaje) {
        System.out.println("Enviando email (Anotaciones): " + mensaje);
    }
}
---
// @Service es una especialización de @Component para lógica de negocio
@Service 
public class NotificadorServiceAnotaciones {

    private final EmailProvider provider;
    private final String prefijoAsunto;

    // INYECCIÓN POR CAMPO (Field Injection) - No recomendada pero común
    // Requisito: @Autowired sobre la variable. No necesita setter/constructor.
    // Beneficio: Escribes muy poco código.
    // Contra: Dificulta los Unit Tests (no puedes instanciar la clase sin Spring).
    @Value("3") // Inyección de primitivo directo
    private int maxReintentos;

    // INYECCIÓN POR CONSTRUCTOR (La recomendada)
    // Requisito: @Autowired sobre el constructor.
    // Beneficio: Puedes testear esta clase con JUnit puro pasando mocks en el constructor.
    @Autowired 
    public NotificadorServiceAnotaciones(EmailProvider provider, 
                                         @Value("[ALERTA]") String prefijoAsunto) {
        this.provider = provider;
        this.prefijoAsunto = prefijoAsunto;
    }

    public void ejecutar() {
        provider.enviar(prefijoAsunto + " Sistema activo. Reintentos: " + maxReintentos);
    }
}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ejemplo" />

</beans>
```
# Task Manager CLI Proyect
### 📋 Especificación del Proyecto

### 1. Objetivo Funcional

Una aplicación de consola para gestionar tareas, pero ahora **los datos deben persistir**. Si cierro la aplicación y la vuelvo a abrir, las tareas creadas anteriormente deben seguir ahí, ya que se leerán de un archivo.

**Las funcionalidades requeridas son:**

1. **Registrar Tarea:** Pedir un ID y una descripción. La tarea nace con estado "activa" por defecto.
    
2. **Listar Pendientes:** Mostrar en consola **solo las descripciones** de las tareas que siguen activas.
    
3. **Completar Tarea:** Pedir un ID. Si la tarea existe, marcarla como "inactiva" (completada). Si no existe, mostrar un mensaje de error amigable.
    
4. **Salir:** Terminar la ejecución.
---

### 2. Reglas Técnicas Actualizadas
Para aprobar este desafío, tu código debe cumplir estrictamente estas restricciones de estilo moderno:

- **Prohibido usar `class` para el modelo de datos:** Debes usar estructuras inmutables.
    
- **Prohibido el `null`:** El repositorio no puede devolver `null`. Debe devolver "cajas" vacías o llenas.
    
- **Prohibido el bucle `for` o `while` para procesar listas:** Debes usar el enfoque declarativo (tuberías de datos) para filtrar y transformar las tareas en el servicio.
    
- **Prohibido concatenar Strings con `+` en el menú:** Debes usar bloques de texto.
    
- **Prohibido el `switch` antiguo:** Debes usar la versión que retorna valores.
    
- **Inferencia de tipos:** No declares tipos obvios a la izquierda de la asignación.
	
- **Persistencia:** Todos los datos deben guardarse en un archivo llamado `tasks.json` en la raíz del proyecto.
    
- **Librería Externa:** Debes agregar la dependencia `com.fasterxml.jackson.core:jackson-databind` a tu proyecto (Maven/Gradle).
    
- **Serialización:** No puedes manipular el archivo `String` manualmente (nada de `split(",")`). Debes usar la clase **`ObjectMapper`** de Jackson para convertir de _Objetos Java_ a _JSON_ y viceversa.
    
- **Manejo de Excepciones:** Debes manejar las `IOException` que puedan surgir al leer/escribir el archivo, preferiblemente capturándolas y lanzando una `RuntimeException` para no ensuciar la interfaz del servicio.
    

---

### 3. Estructura de Clases Requerida

#### A. `Task` (Paquete model)

- Debe ser un **Record**.
    
- Datos: `id` (Integer), `description` (String), `isActive` (boolean).
    
- **Validación:** Debe impedir que se cree una tarea con descripción vacía o nula desde su constructor.

#### B. `TaskRepository` (Paquete repository)
	
- Debe ser un Bean de Spring (`@Repository`).
    
- **Dependencia:** Debe tener un `ObjectMapper` como atributo (puedes instanciarlo ahí mismo o inyectarlo, para simplificar instáncialo como `private final ObjectMapper mapper = new ObjectMapper();`).
    
- **Atributo Archivo:** Debe tener una referencia al archivo, ej: `private final File file = new File("tasks.json");`.
    
- **Lógica de Carga (Constructor o `@PostConstruct`):** Al iniciar, debe verificar si el archivo `tasks.json` existe.
    
    - Si existe: Leer el contenido y convertirlo a una `List<Task>` o `Map` en memoria usando `mapper.readValue(...)`.
        
    - Si no existe: Inicializar la lista/mapa vacío.
        
- **Lógica de Guardado (Método privado `saveToFile`):** Cada vez que se modifique la lista (en el método `save`), debes sobrescribir el archivo JSON con el estado actual de la lista usando `mapper.writeValue(...)`.
    
- **Métodos públicos (`findAll`, `save`, `findById`):** Mantienen la misma firma que antes, pero internamente ahora interactúan con el archivo (leyendo al inicio y escribiendo al modificar).
    

#### C. `TaskService` (Paquete service)

- Debe ser un Bean de Spring (`@Service`).
    
- Debe inyectar el `TaskRepository` vía constructor.
    
- **Método `getActiveTasksDescription`:** Debe usar **Stream API** para:
    
    1. Obtener todas las tareas.
        
    2. Filtrar las activas (`filter`).
        
    3. Extraer solo la descripción (`map`).
        
    4. Retornar una lista de Strings (`toList`).
        
- **Método `completeTask`:** Recibe un ID y devuelve un mensaje (String). Debe usar **`Optional`** para buscar la tarea y, si existe, guardar una copia nueva con `isActive = false`. Si no existe, devolver mensaje de error.
    

#### D. `AppConfig` (Paquete config)

- Clase de configuración de Spring.
    
- Debe indicar a Spring dónde escanear los componentes (`@ComponentScan`).
    

#### E. `Main` (Paquete raíz)

- Clase principal con el método `main`.
    
- Levanta el contexto de Spring (`AnnotationConfigApplicationContext`).
    
- Maneja el bucle del menú (`while`).
    
- Usa **Text Blocks** para imprimir el menú visualmente.
    
- Usa **Switch Expressions** para manejar la opción elegida por el usuario y decidir si continuar o salir.