Una `collection` es un conjunto de elementos sin un orden concreto.
- Son clases e interfaces utilizadas para almacenar, buscar, ordenar y organizar información de manera mas fácil utilizando patrones y métodos estandarizados.

>[! important]
>- `.get(i)` -> Para acceder a una posición de la collection.
>- Son parte del paquete `java.util`

# List *interface*.
Usar clases de `List` cuando lo primoridal sea el orden, puede haber duplicados y puedes acceder mediante el index.
- ## ArrayList *class*
	- Es una lista estandar de elementos.
	- Su tamaño se va a justando a la cantidad de elementos.
	- Mantiene el orden de ingreso y permite duplicados.
- ## LinkedList *class*
	- Contiene métodos adicionales a ArrayList.
	- Es doblemente enlazada. *Creo que quiere decir que se puede recorrer en ambos sentidos e insertar y eliminar datos de cualquiera de las 2 puntas* #duda 
	- Permite operaciones de `insert` y `remove` mas rapidos.
# Set *interface*
Usa clases de `Set` cuando necesites almacenar valores unicos, no acepta repetidos.
- Es una lista de elementos que no contiene repetidos. 
- En el caso de encontrar un repetido deja una instancia unica de ese elemento ? #duda
- Tienen el método `contain()` para validar si existe este elemento en el Set (devuelve true o false).
- ## HashSet *class*
	- Lista desordenada de elementos unicos.
- ## TreeSet *class*
	- Lista ordenada naturalmente (menor->mayor/alfabetico) de elementos unicos.
- ## LinkedHashSet *class*
	- Lista ordenada por insercion de elementos unicos.
## Queue (Colas)
- FIFO (First in, first out)
- ## DeQueue
	- Clase mas habitual. #search 
- ## BlockingQueue #search 
# Map *interface*
Utiliza clases que implementen `Map` cuando necesites almacenar un valor y relacionarlo con una clave.
- Implementa la estructura `clave,valor` 
- ## HashMap *class*
	- Almacena conjunto de `clave,valor` sin un orden especifico.
- ## TreeMap *class*
	- Guarda elementos en formato `clave,valor` de forma ordenada según la clave.
- ## LinkedHashMap *class*
	- Guarda conjuntos `clave,valor` y mantiene el orden de inserción. 

---
>[!question] Todos 
>- [ ] Repasar POO completo.
>	- [ ] Buscar una manera de entender interface y clase que la implementa de una manera mas natural. (que lo entienda un nene de 5 años).
>- [ ] Cual es la diferencia entre `--index` y `index--`. Si le paso valor 13, el primero los resta a 12 antes de entrar al método, el segundo parece que devuelve 13 de igual manera.
>- [ ] Como ordenar una lista de objetos segun un atributo (Collection.sort() + comparator ? Stream.sorted()?)
>- [ ] Existe métodos para transformar un List en un Set ? o se hace mediante un for y agregando elemento por elemento.
# Bibliografia
- https://www.arquitecturajava.com/java-collections-framework-y-su-estructura/
- https://www.w3schools.com/java/java_collections.asp