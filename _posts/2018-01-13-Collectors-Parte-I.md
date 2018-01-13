---
layout: post
title:  Collectors Parte I
category: Java 
description: Primera entrega de las operaciones de reducción sobre Streams definidas en la clase Collectors. 
---
## Introducción

[Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) es una clase incluída en el paquete java.util.stream que implementa un conjunto de operaciones de reducción sobre streams de datos. Algunos ejemplos son la agrupación de valores por una clave común o realizar operaciones de agregación sobre el stream de objetos.

En esta primera parte se van a tratar las funciones más simples que aporta esta clase, las cuales son:

- Contar los elementos de un Stream
- Transformación del Stream en una Lista
- Transformación del Stream en un Conjunto (Set)
- Transformación a una colección
- Obtención del objeto con el valor máximo por campo seleccionado
- Obtención del objeto con el valor mínimo por campo seleccionado

Para presentar los distintos ejemplos partimos de la clase _Coche_ :

```java
class Coche{
    String marca;
    String modelo;
    int caballos;

    public Coche(String marca, String modelo, int caballos){
        this.marca = marca;
        this.modelo = modelo;
        this.caballos = caballos;
    }

    /** getters, setters y toString **/
}
```

Y además partimos de que tenemos el siguiente array de valores ya definido 
para cada ejemplo:

```java
Coche[] coches = { 
    new Coche("SEAT","LEÓN", 90),
    new Coche("SEAT","CORSA",75),
    new Coche("RENAULT","CLIO",120),
    new Coche("FIAT","500",75),
    new Coche("AUDI","A2",105),
    new Coche("AUDI","A3",220),
    new Coche("VW","GOLF",110)
};
```

## Contar los elementos de un Stream
La función public static <T> Collector<T,?,Long> counting() cuenta el número
de elementos que pasan a través del stream y devuelve su resulato como un long.

```java
long numCoches = Stream.of(coches)
                    .collect(Collectors.counting();
System.out.printf("Número de coches: %s", numCoches); // Número de coches: 7
```

## Transformación del Stream en una Lista
La función public static <T> Collector<T,?,List<T>> toList() toma uno a uno
los elementos que procesa el stream y los concatena en una lista.

```java
List<Coche> lista = Stream.of(coches)
                        .collect(Collectors.toList());
System.out.println(lista);
```


## Transformación del Stream en un Conjunto (Set)
La función public static <T> Collector<T,?,Set<T>> toSet() realiza una operación
similar al de lista. Hay que tener en cuenta, que al tratarse de un Set, los
objetos repetidos que se encuentren en el stream solo aparecerán una sola vez en 
el conjunto resultante.

```java
Set<Coche> set = Stream.of(coches).
                        .collect(Collectors.toSet());
System.out.println(set);
```


## Transformación a una colección
Si la colección no se ajusta a nuestras de procesado, porque necesitamos una 
implementación específica o alguna funcionalidad realacionada con la colección
creada, podremos crearla a través de la función 
public static <T,C extends Collection<T>> Collector<T,?,C> toCollection(Supplier<C> collectionFactory)
```java
LinkedList<Coche> lista = Stream.of(coches)
                        .collect(Collectors.toCollection(LinkedList::new));
System.out.println(lista);
```


## Obtención del objeto con el valor máximo en el campo seleccionado
Para el caso en que se necesite encontrar la clase que contiene entre sus atributos el valor más alto
se puede hacer uso de la función public static <T> Collector<T,?,Optional<T>> maxBy(Comparator<? super T> comparator).
Esta función hace uso de un Comparator para poder ordenar los valores de la colección y así obtener el objeto con
el mayor valor en el campo indicado en el comparador.

En este caso, la función devuelve un objeto Optional<Coche>, esto se debe a que si el stream de datos pasados
al Collector fuera vacío, la ejecución de la sentencia asegura una salida y evita excepciones.
```java
Optional<Coche> rapido = Stream.of(coches)
				.collect(Collectors.maxBy(Comparator.comparingInt(Coche::getCaballos)));
System.out.println(rapido.get());
```

## Obtención del objeto con el valor mínimo en el campo seleccionado
El funcionamiento de la función minBy es similar a la función maxBy, salvo que en vez de seleccionar el valor más
alto del stream de datos, escoge el valor más bajo.
```java
Optional<Coche> lento = Stream.of(coches)
				.collect(Collectors.minBy(Comparator.comparingInt(Coche::getCaballos)));
System.out.println(lento.get());
```

En los siguientes post se describirán más operaciones que se pueden realizar sobre streams gracias a la clase Collectors.