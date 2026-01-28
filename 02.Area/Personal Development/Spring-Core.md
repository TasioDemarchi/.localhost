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
