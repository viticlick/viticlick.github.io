---
layout: post
title:  Inmutabilidad de Objetos en Java
category: Java
description: Explicamos qué son los objetos inmutables y su importancia en el desarrollo del software. 
---
Imagina un lugar en donde los programas concurrentes los cuales programar los
hilos es realmente sencillo. La principal complejidad reside en impedir que dos
o más hilos accedan a un mismo recurso a la vez. Muchos pensarán que esto es
una tarea ardua y que requiere de una gran complejidad, pero lejos de la 
realidad, los objetos inmutables nos ayudan a alcanzar nuestro objetivo. Pero,
¿cómo es posible que nos simplifiquen tanto la vida como se dice?, empecemos 
definiendo qué es un objeto inmutable:

> __Los objetos inmutables son aquellos que una vez creados, no pueden ser 
modificados. Por consiguiente, si un objeto es inmutable, sus instancias
son inmutables.__

Como ejemplos de clases inmutables en Java pueden ser 
[String](https://docs.oracle.com/javase/7/docs/api/index.html?java/lang/String.html) 
o [Integer](https://docs.oracle.com/javase/7/docs/api/java/lang/Integer.html).
Ambas clases, una vez creadas, no puede modificarse su
valor interno, y todas aquellas operaciones que se realizan sobre ellas 
devuelven nuevos objetos.

Debido a estas propiedades, los objetos inmutables presentan las siguientes
ventajas:
- Thread safety (Seguridad entre hilos)
- Evitan acoplamiento temporal
- Evitan efectos laterales
- Evitan la mutabilidad de la entidad
- Evitan fallos de atomicidad
- Evitan las referencias a nulos
- Se pueden cachear con facilidad

## Thread safety (Seguridad entre hilos)

La primera razón, y más clara es que un objeto es seguro entre hilos. Lo cual
significa que varios hilos pueden acceder al mismo objeto sin peligro de que 
otro hilo modifique su valor.

Si no existen métodos que puedan modificar el estado del objeto, no importa 
cuantas lecturas se hagan sobre el objeto, y cuantos hilos lo hagan a la vez, 
ya que su estado siempre va a ser el mismo.

## Evitan acoplamiento temporal

Este es un ejemplo de acoplamiento temporal (el código hace dos llamadas HTTP
PUT consecutivas, donde la segunda contiene un cuerpo HTTP):

```java
Request solicitud = new Request("http://midominio.es");
solicitud.method("PUT");
String primera = solicitud.fetch();
solicitud.body("cuerpo=holahola");
String segunda = solicitud.fetch();
```

El código funciona correctamente y es sencillo. Pero deberías recordar que la
_primera_ solicitud debería estar configurada antes que la _segunda_. Si
eliminamos la primera solicitud del código, nosotros no querremos encontrar un
error en su ejecución.

```java
Request solicitud = new Request("http://midominio.es");
// solicitud.method("PUT");
// String primera = solicitud.fetch();
solicitud.body("cuerpo=holahola");
String segunda = solicitud.fetch(); 
```

En esta ocasión, cuando se ejecute el código, se generará una excepción, a 
pesar de no tener ningún error de compilación. Esto es a lo que se le llama un 
**acoplamiento temporal**. Existe información oculta en el código que el 
programador no recuerda. En este caso, hay que recordar que la configuración
de la primera llamada es también utilizada en la segunda.

Obliga a recordar que la segunda solicitud tiene que ir siempre junto a la
primera y ejecutada después de la primera.

Si _Request_ fuese una clase inmutable, el primer ejemplo no funcionaría y 
tendría que ser reescrito así:

```java
final Request solicitud = new Request("http://midominio.es");
String primera = solicitud.method("PUT").fetch();
String segunda = solicitud.method("PUT").body("cuerpo=holahola").fetch();
```

Ahora las dos peticiones no están acopladas. Podemos eliminar la primera con
seguridad y la segunda seguiría funcionando correctamente. Por supuesto puedes
detectar que hay una duplicidad en el código. Pero esto se puede mejorar del 
siguiente modo:

```java
final Request solicitud = new Request("http://midominio.es");
final Request put = solicitud.method("PUT");
String primera = solicitud.fetch();
String segunda = solicitud.body("cuerpo=holahola").fetch();
```

Así, haciendo refactoring, no se rompe nada en el código y obtenemos un 
código limpio de acoplamiento temporal.

## Evitan efectos laterales

Un efecto lateral es causado por una expresión, una llamada a una función, o
la modificación una variable local en una función o procedimiento.

Por ejemplo si añadimos la función:
```java
public String put(Request solicitud) {
    solicitud.method("PUT");
    return solicitud.fetch();
}
```

Si realizamos dos llamadas, una con el método GET y otra con la nueva función

```java
Request solicitud = new Request("http://midominio.es");
solicitud.method("GET");
String primera = this.post(solicitud);
String segunda = solicitud.fetch();
```

En este caso el método _post()_ tiene un _efecto lateral_. Esto hace un cambio
en el objeto mutable solicitud. Estos cambios no son realmente esperados en
este caso. Nosotros esperamos hacer una llamada POST y devolver su cuerpo.
Nosotros no queremos leer la documentación para saber qué pasa en los lodos y si se modifican los estados al haber pasado la clase por argumento.

## Evitan mutabilidad de la entidad

Es común que si tenemos objetos con el mismo estado interno, estos sean el 
mismo. El objeto [Date](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html)
podría ser un ejemplo:

```java
Date fecha1 = new Date(1234l);
Date fecha2 = new Date(1234l);
System.out.println(fecha1.equals(fecha2)); // true
```
Son dos objetos distintos, sin embargo la comprobación del método _equals()_
indica que el valor encapsulado es el mismo. Esto ocurre porque se han 
sobreescrito los métodos _equals()_ y _hashCode()_ de la clase.

Esto conlleva a que si en algún momento se modifica el valor interno de la 
clase, tambien se modifica su valor de identidad:

```java
Date fecha1 = new Date(1234l);
Date fecha2 = new Date(1234l);
fecha2.setTime(4321l);
System.out.println(fecha1.equals(fecha2)); // false
```

Este es un comportamiento esperado cuando se modifica el estado interno de la 
clase, incluso cuando son claves de un mapa:

```java
Map<Date,String> map = new HashMap<>();
Date fecha = new Date();
map.put(fecha, "Hola mi amigo");
fecha.setTime(1234l);
System.out.println(map.containsKey(fecha)); // false
```

Cuando se cambia el estado del objeto _fecha_, no esperamos cambiar su 
identidad. De hecho, no esperamos perder la entrada en el mapa, porque el 
estado de su clave cambie. Sin embargo, eso es exactamente lo que ha pasado.

Cuando se añade un objeto a un mapa, su _hashCode()_ devuelve un valor. Este
valor es el que se utiliza para calcular la posición dentro del mapa interno.
Cuando se realiza la llamada a _containsKey()_, el código hash del objeto ha 
cambiado (porque ha cambiado su estado interno) y el mapa no puede encontrar el
valor en su mapa interno.

Es un cambio que puede parecer raro y complica la depuración debido a la 
mutabilidad de los objetos. Por tanto, evita los objetos mutables como claves
siempre.

## Evitan fallos de atomicidad

Este es un ejemplo:

```java
public class Persona{
    private int hijos;
    private String[] nombreHijos;
    public void setEdad(String hijo){
        hijos++;
        if( hijo == null || hijo == "" ){
            throw new RuntimeException("El hijo tiene que tener un nombre");
        }
    }
}
```

Está muy claro que un objeto de la clase _Persona_ lanzará una excepción si 
se inserta un hijo sin nombre. El valor de _hijo_ se incrementa, mientras que
el conjunto de nombres permanece igual.

La inmutabilidad del objeto previene este problema. Un objeto no se romperá su
estado si la modificación solo se hace en su constructor. El constructor nunca 
fallará, o en su defecto, evitará la instanciación del objeto, pero todo objeto
que cree, será una instancia válida, segura, la cual no cambiará de estado. 

## Evitan las referencias a nulos.
La creación de los objetos mediante sus constructores, fuerza a que en el 
momento de la creación se indiquen todos los parámetros que el objeto pueda 
contener. Si se hacen las comprobaciones de nulos en ese momento, se asegura
que cualquier llamada que extraiga cualquiera de sus campos internos nunca
devuelva un valor a nulo.

## Se pueden cachear con facilidad
Que un objeto sea inmutable implica que su estado no podrá ser modificado, tal y 
como se ha explicado hasta el momento. Por tanto, si se realiza una buena 
estrategia de cacheo, se podrá obtener siempre a través de su _hashCode()_ la 
misma instancia, sin modificaciones, y pudiendo así ahorrar memoria con un pool
de instancias.

