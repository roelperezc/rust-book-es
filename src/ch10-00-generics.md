# Tipos Generic, Traits y Lifetimes

Cada lenguaje de programación tiene herramientas para manejar eficazmente la
duplicación de conceptos. En Rust, una de esas herramientas son los *generics*:
sustitutos abstractos de tipos concretos u otras propiedades. Podemos expresar
el comportamiento de los generics o cómo se relacionan con otros generics sin
saber qué estará en su lugar cuando se compile y ejecute el código.

Las funciones pueden tomar parámetros de algún tipo generic, en lugar de un
tipo concreto como `i32` o `String`, de la misma manera que una función toma
parámetros con valores desconocidos para ejecutar el mismo código en múltiples
valores concretos. De hecho, ya hemos usado generics en el Capítulo 6 con
`Option<T>`, Capítulo 8 con `Vec<T>` y `HashMap<K, V>`, y Capítulo 9 con
`Result<T, E>`. ¡En este capítulo, explorará cómo definir sus propios tipos,
funciones y métodos con generics!

Primero, revisaremos cómo extraer una función para reducir la duplicación de
código. Luego usaremos la misma técnica para hacer una función generic a
partir de dos funciones que difieren solo en los tipos de sus parámetros.
También explicaremos cómo usar tipos genéricos en definiciones de structs y
enums.

Entonces aprenderás cómo usar *traits* para definir el comportamiento de una
manera genérica. Puedes combinar traits con tipos generics para restringir un
tipo generic para que acepte solo aquellos tipos que tienen un comportamiento
particular, en lugar de cualquier tipo.

Finalmente, discutiremos *lifetimes*: una variedad de generics que le dan al
compilador información sobre cómo se relacionan las referencias entre sí.
Lifetimes nos permiten darle al compilador suficiente información sobre los
valores prestados para que pueda garantizar que las referencias serán válidas
en más situaciones de las que podría sin nuestra ayuda.

## Eliminando la duplicación extrayendo una función

Los generics nos permiten reemplazar tipos específicos con un marcador de
posición que representa múltiples tipos para eliminar la duplicación de código.
Antes de sumergirnos en la sintaxis de los generics, veamos primero cómo
eliminar la duplicación de código de una manera que no involucre tipos
genéricos extrayendo una función que reemplace valores específicos con un
marcador de posición que represente múltiples valores. ¡Luego aplicaremos la
misma técnica para extraer una función genérica! Al observar cómo reconocer el
código duplicado que puede usar en una función, comenzará a reconocer el
código duplicado que puede usar en los generics.

Comenzamos con un corto programa en el Listado 10-1 que encuentra el número
más grande en una lista.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

<span class="caption">Listing 10-1: Encontrando el mayor número en una lista de
números</span>

Almacenamos una lista de enteros en la variable `number_list` y colocamos una
referencia al primer número de la lista en una variable llamada `largest`.
Luego iteramos a través de todos los números de la lista, y si el número
actual es mayor que el número almacenado en `largest`, reemplazamos la
referencia en esa variable. Sin embargo, si el número actual es menor o igual
al número más grande visto hasta ahora, la variable no cambia, y el código
pasa al siguiente número de la lista. Después de considerar todos los números
de la lista, `largest` debería hacer referencia al número más grande, que en
este caso es 100.

Ahora se nos ha encargado encontrar el número más grande en dos listas de
números. Para hacerlo, podemos duplicar el código en el Listado 10-1 y usar la
misma lógica en dos lugares diferentes en el programa, como se muestra en el
Listado 10-2.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

<span class="caption">Listing 10-2: Código para encontrar el mayor número en 
*dos* listas de números</span>

Aunque este código funciona, duplicar el código es tedioso y propenso a errores.
También tenemos que recordar actualizar el código en varios lugares cuando
queremos cambiarlo.

Para eliminar esta duplicación, crearemos una abstracción definiendo una
función que opera en cualquier lista de enteros que se pase en un parámetro.
Esta solución hace que nuestro código sea más claro y nos permite expresar el
concepto de encontrar el número más grande en una lista de forma abstracta.

En el Listado 10-3, extraemos el código que encuentra el mayor número en una
función llamada `largest`. Luego llamamos a la función para encontrar el mayor
número en las dos listas del Listado 10-2. También podríamos usar la función
en cualquier otra lista de valores `i32` que podríamos tener en el futuro.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

<span class="caption">Listing 10-3: Código abstracto para encontrar el número
mayor en dos listas</span>

La función `largest` tiene un parámetro llamado `list`, que representa cualquier
slice de valores `ì32` que podriamos pasar a la función. Como resultado, cuando
llamamos a la función, el código se ejecuta en los valores específicos que
pasamos.

En resumen, estos son los pasos que tomamos para cambiar el código del Listado
10-2 al Listado 10-3:

1. Identificar código duplicado.
2. Extraer el código duplicado en el cuerpo de la función y especificar las
   entradas y salidas de ese código en la firma de la función.
3. Actualizar las dos instancias de código duplicado para llamar a la función
   en su lugar.

A continuación, usaremos estos mismos pasos con los generics para reducir la
duplicación de código. De la misma manera que el cuerpo de la función puede
operar en una `list` abstracta en lugar de valores específicos, los generics
permiten que el código opere en tipos abstractos.

Por ejemplo, digamos que teníamos dos funciones: una que encuentra el mayor
elemento en un slices de valores `i32` y otra que encuentra el mayor elemento
en un slice de valores `char`. ¿Cómo eliminaríamos esa duplicación? 
¡Averigüémoslo!
