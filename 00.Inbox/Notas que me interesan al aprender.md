## Enums en entidades
- Al usar ENUM en entidades es buena practica agregar la anotación `@Enumerated(EnumType.STRING)`.
	- **Para que sirve ?** Para que en la base de datos se guarde el valor en texto (String) y no un indice.
			Por default JPA utiliza `EnumType.ORDINAL` que guarda los indices 0 para el primer valor, 1 para el segundo y así sucesivamente. Si se agrega un nuevo valor al inicio del enum, las entidades almacenadas en la base va a estar apuntando a un valor incorrecto.

>[!FAQ] Dudas
>Porque se utiliza el `final` para los atributos de las clases serivce y configuration
>Entender el patron `Builder`.
>Las clases de Request y Response van en la carpeta DTO ? Porque en vez de usar una clase completa no usar un Record ?
>### Clase `JwtService`
>Como funciona el el método `extractUsername` cuando no se especifica que lo que tiene que buscar es el username ?
>### Clase `JwtAuthenticationFilter`
>Porque se le agrega la anotación `@NonNull` a los parámetros del método `doFilterInternal`?
>