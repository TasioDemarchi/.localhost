### Spring-Security 
- Es el framework encargado de mantener las aplicaciones Spring segurizadas
- Esta segurización se realiza por medio de una cadena de filtos, los cuales se encuentran por delante del código de negoció de la aplicación. Frenando los ataques a la aplicación antes de que se ejecute código de negocio.
- Se encarga de 2 principales apartados:
	- **Autenticación:** Corroborar la validez del usuario.
		- Login/Registro
			- Utiliza el `PasswordEncoder` para hashear la contraseña y almacenarla en la base de datos.
			- Valida la el hash generado a partir de la contraseña ingresada por el usuario en el login, contra el hash guardado en la BD para permitir o denegar el acceso a la aplicación.
			- Si el login es valido, se genera un JWT que se envia al frontend para almacenarlo en el `localStorage` del navegador, el cual va a ser utilizado en la parte de autorización.
		- Peticiones luego del login
			- Utiliza el JWT para consumir endpoints de la parte del backend. 
			- Este JWT es enviado en cada petición como un Header Authorization = Bearer JWT.
			- El JWT tiene una estructura divida en 3
				- **Header:** Donde se define el algoritmo utilizado. Información visible en [jwt.io](https://www.jwt.io/)
				- **Payload:** Información que se necesita del usuario (username, rol, expiracion del Token, ...). Información visible en [jwt.io](https://www.jwt.io/)
				- **Signature:** La firma del token, la cual se construye con la información del Header + Payload + la clave secreta (SECRET_KEY). Información que no se revela.
			- Spring-security realiza la validación de la siguiente manera para autorizar:
				- Spring genera el **Signature** con los datos del header y el payload y envia el JWT completo en el login.
				- El navegador almacena este JWT en el `localStorage` para futuras peticiones.
				- En las peticiones interceptadas por Spring-Security se realiza la validación del JWT de la siguiente manera:
					- Se toma el header y payload recibidos y se genera nuevamente el **Signature**.
					- Si el **Signature** corresponde con el recibido (original), entonces se resuelve la petición.
					- Si el **Signature** es distinto al recibido, entonces la solicitud se rebota enviando un error al cliente.
					- *Explicacion tecnica:*
						-  En el login se genero el JWT `header.user.AAA` creando el **Signature** (firma de token) de la siguiente manera `HMACSHA256( Header + "." + user, "MiSecreto" )`.
						- En la petición el filtro de Spring-security recibe `header.admin.AAA` (JWT modificado por atacante).
						- Para validar Spring genera nuevamente la **signature** con la información del JWT recibido `HMACSHA256( Header + "." + user, "MiSecreto" )` y el resultado es que el **signature** es ZZZ.
						- Compara ambos signature (AAA != ZZZ) y al ver que no coinciden la firma revoca la consulta.
	-  **Autorización:** Validar que el usuario pueda realizar X acción.
		- Valida el rol que posee el usuario antes de cada acción para corroborar que tenga permisos sobre ella.
		- Esta validación se puede hacer de 2 formas:
			- Confiando en el dato de JWT.
				- Se valida el **Signature**, si es correcto se extrae el rol.
				- **Ventajas:** Es rapido, no requeire consulta a BD.
				- **Desventajas:** No esta actualizado. Si se le cambia el rol, pero el JWT no expiro, el seguira siendo el rol anterior.
			- Consulta a la BD.
				- Se valida el **Signature**, si es correcto se realiza una consulta a la BD para conocer el rol de este usuario.
				- **Ventajas:** Seguridad inmediata. El cambio de rol se ara de manera directa.
				- **Desventaja:** Por cada petición se realiza una consulta a la base de datos.
		

- [ ] Filtros de seguridad de spring
- [ ] Servidor Stateless
	- Es lo contrario a Stateful:
		- Antes al logearte el servidor guardaba una info de la sesion en su memoria RAM y devolvia una cookie (`JSESSIONID`) con dicho valor. Al realizar una nueva petición, se enviaba este valor y el servidor buscaba en la memoria de quien viene la petición. Problema: Si tenes 3 servidores sobre un balanceadro, dificil saber quien envia la petición.
		- Stateless: El servidor no almacena información, toda la información necesaria para una consulta va en la request. De esta manera el servidor resuelve validaciones por cada petición, permitiendo escalar el servidor a gusto.
- [ ] @EnableWebSecurity
- [ ] que es CSRF
	- Ciberataque el cual consiste en que el atacante defina una actividad maliciosa dentro de alguna actividad de una pagina, la cual utilzia las cookies almacenadas en el navegador para realizarlo. Ejemplo: Utiliza las cookies de sesion del banco para realziar una transfrenecia, sin que el usuario se de cuenta. 
- [ ] La tabla de Usuarios tiene alguna estructura predefinida ? 
[[Untitled.canvas]]

Las reglas de Spring-Security se leen de arriba a abajo. En tiempo de ejecución Spring-security busca la primera regla que coincida/afecte a la petición y la ejecuta sin ver las reglas consiguientes. Por lo que se debe poner las reglas mas especificas en el principio y las mas generales en el final

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Deshabilitamos CSRF para REST simple
            .authorizeHttpRequests(auth -> auth
            // Para una consulta sobre el endpoint "/public/hello" le pedira autorización ya que la Regla A coincide con la ruta de petición, aunque la Regla B sea mas especifica.
                .anyRequest().authenticated()      // Regla A
                .requestMatchers("/public/**").permitAll() // Regla B
            )
            .httpBasic(Customizer.withDefaults()); // Usamos Basic Auth

        return http.build();
    }
}
```

## Pasos de Spring-Security para la Autenticación
1. Llega la petición HTTP
2. Spring-Security la intercepta mediante su filtro.
3. Valida que regla de las definidas en su filtro debe aplicar a esta petición (suponemos que solicita autenticación).
4. Mediante la implementación `loadUserByUsername` se obtienen los datos del usuario de la BD con ayuda del repositorio.
5. Se mapea los datos al tipo de dato `UserDetails`, el cual es manejado por Spring-Security.
6. Si existe una implementación de **PasswordEncoder** Spring validara que la contraseña obtenida en el UserDetails este en un formato **Hash** segun el algoritmo especificado.
7. Spring validara los **Hashs** mediante `matches(inputPass, dbHash)`.
8. Si devuelve `true`, Spring-Security dejara pasar la petición para que se resuelva.

---
## Explicación código Spring-Security
### 1. Parte: Filtros
>[!FAQ] Dudas
> - [ ] El bean definido en el método `securityFilterChain` Spring lo buscara para las reglas, por su nombre o porque esta en la clase anotada por `@EnableWebSecurity` ?.
> - [ ] Al anotar rutas con el método `hasRole("ADMIN")` tambien se le solicita autenticación por medio de JWT aunque no se utilice el método `.authenticated()` ?.
> - [ ] Necesito una mejor explicación de Stateless. Es posible que en proyecto legacy el backend usaba estados ? porque ya no se usan (ventajas y desventajas) ?


