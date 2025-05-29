author: Erick Varela
summary: Guía de aprendizaje 01
id: g05
categories: Databases
environments: Databases
status: Published
feedback link: mailto:e

# Funciones de agregacion y Subconsultas

## Funciones de agregacion

Las funciones de agregación son funciones integradas en SQL que permiten aplicar operaciones matemáticas sobre conjuntos de valores, típicamente agrupados según una o más columnas de forma tal que los datos se operan en base a una regla de agrupacion que se tiene que definir de forma predeterminada.

Un ejemplo simple es calcular el total de todas las facturas de un cliente en especifico. Asumamos que tenemos la siguiente lista:

| **id_cliente** | **Total_factura** |
|----------|----------|
| 1 | 3.4 |
| 1 | 5.2 |
| 1 | 1.0 |
| 2 | 3.6 |
| 3 | 30.5 |
| 2 | 7.4 |

(tabla 1)

A nosotros, como encargados de bases de datos, nos podria interesar saber cuanto ha gastado cada cliente en total con todas las facturas. Para eso tenemos que sumar el **Total_factura** para cada cliente, es decir, sumar todos los **Total_factura** para el **id_cliente** 1, hacer otro calculo para el **id_cliente** 2, y otro para **id_cliente** 3. En general, esperamos una lista de la siguiente manera

|**id_cliente**|**Total_factura**|
|----------|----------|
| 1 | 9.6 |
| 2 | 11.0 |
| 3 | 30.5 |

(tabla 2)

<aside class="positive">
En esencia, lo que realizamos aqui es agrupar las sumas de Total_factura por cada id_cliente, es decir, le sumamos los total_factura a cada cliente que le correspondia 
</aside>

Para realizar este proceso nos apoyaremos de funciones de agregacion, es decir, invocaremos una funcion que nos permita sumar los **Total_factura** para cada cliente.

Para ello usaremos la funcion SUM(), que nos permite hacer justo esto, pero antes, estudiaremos que funciones de agregacion tenemos.

<aside class="positive">
Cabe recalcar que debido a la naturaleza de las funciones de agregacion que realizan procedimientos matematicos, solo se pueden usar sobre columnas con valores numericos generalmente, aunque existen algunas excepciones como COUNT() 
</aside>

## Tabla de funciones de agregacion

|**Funcion**|**Descripcion**|
|----------|----------|
| COUNT() | Realiza un conteo de cada instancia de valores no nulos. Devuelve un valor numerico con el resultado |
| SUM() | Realiza la suma de todos los valores de una columna agrupado por una o varias columnas definidas por el usuario|
| AVG() | Calcula el promedio de la columna definida, es decir, suma los valores agrupados y los divide por la cantidad de numeros sumados |
| MAX() | Devuelve como valor numerico el numero mas grande de la columna para cada columna agrupada |
| MIN() | Devuelve como valor numerico el numero mas pequeño de la columna para cada columna agrupada |
| STDEV() | Devuelve la desviacion estandar de la columna para cada columna agrupada |

## Uso de funciones de agregacion

### GROUP BY

Cuando se utilizan funciones de agregacion en una consulta SQL, como vimos con anterioridad, tenemos que saber cómo agrupar los datos para que la función opere correctamente. Para esto, SQL Server nos da el comando **GROUP BY**

**GROUP BY** es una palabra reservada la cual, a partir de ella, nos permite declarar bajo que columnas es realizara la agrupacion al utilizar una funcion de agregacion. En otras palabras, si utilizamos SUM() nos interesa agrupar, por ejemplo, por id_cliente, por lo que en el GROUP BY debemos declarar que la agrupacion se realizara sobre dicha columna

```sql
SELECT id_cliente, SUM(Total_factura) 
FROM Ejemplo
GROUP BY id_cliente
ORDER BY id_cliente
```

La consulta anterior nos devolvera la tabla 2, es decir, la agrupacion de las sumas por id_cliente. 

<aside class="positive">
Cuando se trabaja con funciones de agregacion, se vuelve muy importante definir correctamente las columnas que se solicitan durante la consulta: los datos de estas deberian ser unicos, pues si hay columnas entre ellas con varios valores la agrupacion podria generar datos incorrectos o incompletos, por lo que al trabajar con estas lo optimo es solicitar unicamente lo minimo necesario para darle sentido a la consulta.
</aside>

Finalmente, al usar funciones de agregacion, todas las columnas que se soliciten en las consulta SQL y que se encuentren fuera de las funciones de agregacion deben ser declaradas nuevamente en la seccion de GROUP BY, si no, la consulta SQL devolvera un error.

**Ejemplo 1**: Queremos ver UNICAMENTE cual es la categoria que mas ha recaudado para los proyectos

```sql
SELECT titulo, id_categoria, SUM(Precio) 'Total_categoria' FROM Proyecto
GROUP BY id_categoria, titulo
ORDER BY id_categoria
```
<aside class="positive">
Porque la anterior consulta NO devuelve los datos de forma correcta/como nos interesan? Como lo arreglarias?
</aside>

**Ejemplo 2**: Queremos ver cuanto han recaudado los lideres de proyecto de forma descendente

```sql

SELECT E.id, E.nombre, SUM(P.Precio) 'Total de proyectos' FROM Empleado E
INNER JOIN Proyecto P ON E.id = P.id_lider
GROUP BY E.id, E.nombre
ORDER BY SUM(P.Precio) DESC

```

### HAVING

Cuando se trabaja con funciones de agregacion, se nos puede dar el caso que queramos colocar condiciones a dichas funciones. Por ejemplo, que solo nos interesen aquellos datos que, al ser procesados por una funcion de agregacion, superen el valor de 5 (>5). Generalmente, una condicion de este tipo se colocaria en un WHERE, pero SQL server es incapaz de procesar funciones de agregacion durante dicha sentencia ya que WHERE se tiene que declarar antes que GROUP BY, por lo que durante la lectura del WHERE los datos aun no estan agrupados, asi que nos tendremos que apoyar de la orden **HAVING**

**HAVING** Funciona de la misma manera que WHERE, pero esta tiene que declararse despues de GROUP BY, por lo que ella si es capaz de procesar funciones de agregacion y por ende nos permite poner condicionales sobre estas.

**Ejemplo 1**: Queremos modificar el Ejemplo 2 para que solo nos devuelva aquellos lideres que han recaudado mas de 700$.

```sql

SELECT E.id, E.nombre, SUM(P.Precio) 'Total de proyectos' FROM Empleado E
INNER JOIN Proyecto P ON E.id = P.id_lider
GROUP BY E.id, E.nombre
HAVING SUM(P.Precio) > 700
ORDER BY SUM(P.Precio) DESC

```

**Ejemplo 2**: Queremos saber cuantos empleados hay por departamento bajo ciertas condiciones: Primero, el nombre del departamento debe tener en alguna del mismo 'as'; Segundo, que solo nos devuelva aquellos departamentos con mas de 5 empleados.

```sql

SELECT D.nombre, COUNT(E.id) 'Total de empleados' FROM Empleado E
INNER JOIN Departamento D ON E.id_puesto = D.id
WHERE D.nombre LIKE '%as%'
GROUP BY D.nombre
HAVING COUNT(E.id) >= 5
ORDER BY COUNT(E.id)

```

<aside class="positive">
Como se puede apreciar en el ejemplo 2, somos capaces de combinar las ordenes de WHERE y HAVING para realizar condicionales complejas o de varios tipos. Analice entonces, en que momento deberia usar WHERE y HAVING?
</aside>

## Subconsultas

Las Subconsultas, o SubQueries, son, como el nombre indica, consultas anidadas dentro de:
 - INSERT
 - UPDATE
 - DELETE
 - SELECT

Basicamente, una Subquery es una consulta independiente que funciona por si misma dentro de otra instruccion SQL y que esta es capaz de usar los resultados de la Subquery para su proceso.

Para una mejor comprension del concepto y apreciacion de su utilidad, veamos el siguiente ejemplo:

**Ejemplo 1:** Tenemos que actualizar la columna Estrella de la tabla empleado, la cual representa si este empleado es un lider y ademas ha generado mas de 700$ como ganancias de proyectos, a verdadero (True).

Para resolver el Ejemplo 1, necesitamos saber cuales empleados son aquellos que cumplen la condicion de ser lideres y ademas haber generado mas de 700$ en ganancias de proyectos. Esta es una consulta que ya resolvimos con anterioridad:

```SQL
SELECT E.id, E.nombre, SUM(P.Precio) 'Total de proyectos' FROM Empleado E
INNER JOIN Proyecto P ON E.id = P.id_lider
GROUP BY E.id, E.nombre
HAVING SUM(P.Precio) > 700
ORDER BY SUM(P.Precio) DESC
```

Esta consulta nos devuelve cuatro empleados:
 - Id 17, Nombre Leoncio
 - Id 3, Nombre Susanita
 - Id 20, Nombre Eva Maria
 - Id 1, Nombre Tomasa

Sabiendo estos datos, podriamos vernos inclinados a usarlos para resolver nuestro ejemplo de la siguiente manera

```SQL

UPDATE EMPLEADO SET Estrella = 1 WHERE id = 17 OR id = 3 OR id = 20 OR id = 1

```
La sentencia actualizara, de forma correcta, la columna estrella a verdadero (1) a los empleados que han generado mas de 700$. Sin embargo, esta solucion nos genera dos posibles problemas:
 - Que pasaria si tenemos mas de 200 empleados que cumplen la condicion? Deberiamos colocar el id de esos 200 empleados en la condicion WHERE, lo cual nos generaria una solucion enorme e ineficiente para nosotros
 - Esta solucion es valida por el momento, pero si en un futuro nos solicitan hacer la actualizacion nuevamente y ahora hay mas empleados (o menos) que cumplen la condicion tendriamos que buscar el id de estos y modificar la solucion

Como podemos aprecia, nuestra solucion, aunque correcta es ineficiente y no es replicable para el futuro. Para este tipo de problemas, sin embargo, nos podemos apoya de Subqueries para resolverlo de manera tal que resulte en una solucion corta y que ademas sea dinamica, es decir, aplicable para cualquier caso.

La solucion, usando SubQueries, es la siguiente:

```SQL
UPDATE EMPLEADO SET Estrella = 1 WHERE id IN 
(SELECT E.id FROM Empleado E
INNER JOIN Proyecto P ON E.id = P.id_lider
GROUP BY E.id, E.nombre
HAVING SUM(P.Precio) > 700)
```

Primero, centrandose en la SubQuery, esta es la misma SubQuery que usamos para saber los empleados que han generado mas de 700$ con la modificacion de que unicamente devuelve la lista de Ids de los mismos. El resultado de esta columna es una tabla de la siguiente forma:

|**id**|
|------|
|1|
|3|
|17|
|20|

Teniendo esta lista, podemos modificar nuestra condicional WHERE del UPDATE apoyandonos de la palabra reservada IN, que, como recordamos, cumple una condicion si los datos de la columna que estamos verificando se encuentran dentro de una lista de datos. En esencia, las modificaciones que hicimos le permite al UPDATE verificar los datos de la columna id si existen en la lista que se genera al ejecutar la SubQuery.

De forma similar, podemos usar esta estructura para las sentencias de DELETE, verificando una lista de elementos que queramos eliminar de forma dinamica, sin embargo, uno de los mayores usos de las SubQueries vienen al usar SubQueries dentro de consultas SELECT.

**Ejemplo 2:** Conseguir el promedio de lo ganado por todas las areas de proyectos, tomando en cuenta que el precio total de un proyecto es igual a el precio del mismo mas el precio agregado del area al que pertenece.

Viendo la definicion del ejercicio, podemos vernos inclinados a primero sumar los precios de los proyectos mas el precio de las areas y luego sacar el promedio, de una manera similar a esta:

```sql

SELECT AVG(SUM(P.Precio + A.precio)) 'Precio Total' FROM Proyecto P
INNER JOIN Categoria C ON P.id_categoria = C.id
INNER JOIN Area A ON A.id = C.id_area

```

Sin embargo, SQL presenta una fuerte restriccion a estos casos: No se pueden tener funciones agregadas anidadas, es decir, no podemos hacer AVG(SUM()), o SUM(COUNT()), etc. Eso sin mencionar que aun si se pudiera esa solucion no presenta lo que necesitamos, ya que necesitamos especificamente, primero las ganancias de cada area y luego el promedio de las mismas. Es en estos casos en que nos podemos apoyar, nuevamente, de SubQuery:

```sql

SELECT AVG(SQ.[Precio Total]) 'Promedio' FROM 
(SELECT C.nombre, SUM(P.Precio + A.precio) 'Precio Total' FROM Proyecto P
INNER JOIN Categoria C ON P.id_categoria = C.id
INNER JOIN Area A ON A.id = C.id_area
GROUP BY C.nombre) SQ

```

Lo que podemos apreciar: La SubQuery nos devuelve una lista de las sumas del total de las areas, una tabla con el siguiente formato:

| #  | Nombre              | Precio Total |
|----|---------------------|--------------|
| 1  | Análisis Financiero | 937.1        |
| 2  | Ciberseguridad      | 3721.4       |
| 3  | Contabilidad        | 810.3        |
| 4  | Design Thinking     | 1125.5       |
| 5  | Liderazgo           | 3310.5       |
| 6  | Scrum               | 1082.8       |

Con esos datos, ahora tenemos que sacar el promedio de la columna Precio Total, por lo que construimos una consulta y nuestra fuente de datos (FROM) va a ser la subconsulta. Es decir, la Subconsulta nos va a dejar acceder a los datos de esta como que de una tabla se tratase, lo cual nos permite hacer todos los procedimientos que ya conocemos, como condicionales, uniones con JOIN, usar funciones agregadas, etc. Sabiendo que Precio Total tiene los datos que nos interesan, lo ultimo que tenemos que hacer hacer un promedio de dicha columna y tendremos nuestras respuesta final.

<aside class="positive">
Al usar consultas como fuentes de datos, es importante que todas las columnas tengan un nombre, ya que ese es el nombre que usaremos para referirnos a ellas fuera de la SubQuery. Para columnas normales no hay problema, pero columnas calculadas deben llevar un alias si queremos referenciarlas en algun futuro.
</aside>

Despues de ver estos casos, podemos hacer ciertas conclusiones sobre las SubQueries.

## Usos de Subconsultas

 1. Podemos usar SubQueries como que de una lista se tratasen, o un valor unico, para usarlas dentro de condionales
 2. Podemos usarlas como fuentes de datos dentro de otras consultas para usar los resultados de la SubQuery como una tabla
 3. Todas las columnas de una SubQuery necesitan un nombre si van a ser referenciadas fuera de la misma
 4. Una SubQuery debe funcionar de manera independiente, es decir, ella sola debe poder ser ejecutada y devolver datos
 5. Podemos anidar todas las SubQueries que sean necesitadas, pero se debe tomar en cuenta que a mayor anidacion de SubQueries peor es el rendimiento de una consulta
 6. Si un problema puede resolverse sin necesidad de subconsultas, es recomendable optar por esa solución, ya que generalmente ofrece mejor rendimiento y menor complejidad.
 7. Casos en los que se necesite usar funciones de agregacion sobre funciones de agregacion son problemas que se resuelven con SubQueries
 8. Casos donde se necesiten dato/s de forma dinamica para sentencias de SELECT, UPDATE, INSERT y DELETE son problemas que se resuelve con SubQueries
 9. Dentro de una consulta SQL se puede colocar SubQueries en:
    - La seccion de columnas (Sin embargo, aqui lo unico que puede devolver la consulta es una celda (un solo valor) ya que si no esta devolvera un error)
    - Como fuente de datos despues del FROM (Incluso podemos aplicarle JOINs a una SubQuery ya sea a una tabla que ya poseamos o incluso a otra SubQuery)
    - En condicionales WHERE y HAVING

## Ejercicios

Siga las instrucciones del instructor encargado para la realizacion de los ejercicios. La solucion de estos debera ir con el siguiente formato:

```sql

--Ejercicio (numero del ejercicio)
/*
Solucion del ejercicio
*/
--Ejercicio (numero del ejercicio)
/*
Solucion del ejercicio
*/
--Ejercicio (numero del ejercicio)
/*
Solucion del ejercicio
*/

```

En el script unicamente debe ir la solucion de los ejercicios, sin el script de generacion de la base de datos. A su vez, se calificara que la solucion del ejercicio tiene que devolver una funcion exactamente igual a la solucion propuesta. Caso contrario, el ejercicio se considerara como erroneo.

## Sobre los autores de esta guía práctica

Departamento de Electrónica e Informática, Universidad Centroamericana José Simeón Cañas, La Libertad, El Salvador.

**Versión de este documento:** Versión 1.05, 2025.

| **Versión** | **Año** | **Autores** |
|----------|----------|----------|
| **1.00** | 2025 | Christian Alejandro Paz (00132720@uca.edu.sv) |

![license](./license.jpg) This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).