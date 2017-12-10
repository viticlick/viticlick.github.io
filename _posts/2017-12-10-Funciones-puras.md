---
layout: post
title:  Funciones Puras
category: java
description: En el siguiente post explicamos qué es una función pura.
---
Las funciones puras son utilizadas en el paradigma de la programación funcional y proporcionan un mecanismo de confianza para los programas concurrentes. Pero, ¿qué significa una _"función pura"_?

Para entrar en materia, comenzaremos definiendo qué es una función.

## ¿Qué es una función?
Una definición matemática podría ser:

> Una función es una relación por la cual cada valor procedente de un conjunto de datos ordenados se corresponde exactamente con un único valor de un segundo conjunto de valores ordenados.

La manera habitual de denotar una función es:

     f: A -> B
        a -> f(a)

De un modo más llano, lo que esto significa es que dada una función *__f__*  para el mismo conjunto de valores de entrada *__A__* , siempre se va a generar un mismo valor de salida *__B__*. Esta operación también es llamada _operaciónes de mapeo o mapping_. 

Por ejemplo la función _Math.max(int,int):int_.

```java
Math.max(3, 9); // 9    
```
 En este ejemplo la función _Math.max()_ recibe los enteros _3 y 9_ como valores de entrada, y retorna el valor _9_ ya que es el máximo de los dos. 

 En álgebra las funciones son nombradas de la siguiente forma:

    f(x) = 3x

Lo que significa que se declara la función _f_ que recibe el argumento _x_ y multiplica su valor por 3. En álgebra esto significa que si a la función _f_ le pasamos el valor _3_, el método devolverá el valor _9_.

    f(3) -> 9

Esta misma función en Java 8 o superior sería del siguiente modo:

```java
Function<Integer,Integer> triple = x -> x * 3;
```
## Funciones puras

Una **función pura** es aquella que:
- Dada la misma entrada, siempre devolverá el mismo valor
- No produce ningún efecto lateral

Esta condición favorece el patrón _KISS (Keep It Simple Stupid)_ o traducido, _mantenlo simple
estúpido_. La función debería ser el bloque de código más pequeño y reusable que un programador
debería crear en sus programas. Esta propiedad además hace sencilla la verificación de la
corrección de la función.

### Dada la misma entrada, siempre devolverá el mismo valor

Sobre esta propiedad es importante destacar el **siempre** sobre el enunciado. Cuando las aplicaciones comparten objetos entre distintos procesos o realizan tareas asíncronas, corren el riesgo de que el estado de un objeto varíe con el tiempo. 

Cuando una función no asegura que siempre devuelve el mismo valor para el mismo conjunto de valores de entrada, se dice que es **no determinista**, y este comportamiento se da cuando se cumplen las siguientes partes de la ecuación:

    no determinismo = concurrencia de procesos + estado mutable

#### Concurrencia
Que según la wikipedia, podemos completar con:
> En ciencias de la computación, concurrencia es una propiedad de los sistemas en la cual los
procesos de un cómputo se hacen simultáneamente, y pueden interactuar entre ellos. Los cálculos
(operaciones) pueden ser ejecutados en múltiples procesadores, o ejecutados en procesadores
separados físicamente o virtualmente en distintos hilos de ejecución

#### Mutabilidad
En este caso la wikipedia nos indica que un objeto inmutable es el:
> objeto cuyo estado no puede ser modificado una vez creado.

La mutabilidad de los objetos trae consigo una serie de problemas que aparecen cuando se resuelve
un problema de forma concurrente. Los problemas relacionados con la mutabilidad se dan cuando dos
hilos de ejecución hacen uso de un mismo objeto o zona de memoria del computador y modifican
valores usados en los bloques de código. La alteración de uno de estos valores puede hacer que el
resultado final del método difiera del valor esperado.

### No produce ningún efecto lateral

Para que una función sea denominada pura, es indispensable que no tenga ningún efecto lateral.
Esto se traduce en que la llamada a la función no puede alterar el estado de ningún objeto
distinto al valor devuelto a la función ni tampoco realice ningua operación de entrada/salida en 
el sistema.

Si la función modifica el valor de alguno de los parámetros de entrada, se dice que es _impura_ y por lo tanto no cumple la condición de evitar efectos laterales.

El proceso de realizar alguna operación de entrada/salida en el sistema también es considerado un
efecto lateral, ya que durante el proceso de I/O puede generar algún tipo de excepción no
controlada, y por tanto, provocando que la función no devuelva el valor correspondiente.

En futuros posts trataremos cómo las funciones puras son un pilar de la programación funcional y ayudan a construír programas más eficientes y sencillos.