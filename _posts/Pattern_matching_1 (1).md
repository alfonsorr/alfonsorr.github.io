Esto es para la versión de jupyter, permitimos que nos muestre los warnings de compilación que tenga un bloque de código. Tambíen indicamos que nos muestre los warnings por código inalcanzable. Pero shhh, son spoilers del post.


```scala
println("start")
```

    start



```scala
interp.configureCompiler(_.settings.nowarn.value = false)
```


```scala
interp.configureCompiler(_.settings.warnDeadCode.value = true)
```

## ¿Que es el pattern matching?

Una estructura de control, pensada para comprobar si un elemento cumple ciertas condiciones. Si no la conocías antes es similar en sintaxis a un `switch`, pero nos permite mayor precisión.
Para aplicarlo solo necesitamos poner a continuación del elemento sobre el que queremos aplicarlo, la palabra reservada `match` e indicar cada uno de los casos que nos interesan.

Comencemos con uno ejemplo sencillo:


```scala
val stringExample = "hola"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case _ => "???"
}
```




    [36mstringExample[39m: [32mString[39m = [32m"hola"[39m
    [36mres3_1[39m: [32mString[39m = [32m"saludos"[39m



Vamos a describir que está ocurriendo aquí: tenemos nuestro valor asignado y empezamos el matcheado. En este caso, queremos comprobar si el valor a procesar es igual al string "hola". En caso de ser correcto, se ejecutará el codigo de su parte derecha, si no, pasará al siguiente caso y repetirá lo misma comprobación hasta que uno coincida. Podéis ver que el último caso, se representa solo con `_`, esta es la forma en scala de indicar "cualquier otro caso", por lo que si ninguno de los anteriores ha sido satisfactorio, sabemos que siempre ejecutará la parte derecha de este código.

Esta es la principal diferencia respecto a `switch`. En el patter matching únicamente ejecutará el primer trozo de código que satisfaga la condición, por lo que el orden que indiquemos importa.


```scala
val stringExample = "hola"
stringExample match {
    case _ => "???"
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
}
```

    cmd4.sc:3: patterns after a variable pattern cannot match (SLS 8.1.1)
        case _ => "???"
             ^cmd4.sc:4: unreachable code due to variable pattern on line 3
        case "hola" => "saludos"
                       ^cmd4.sc:5: unreachable code due to variable pattern on line 3
        case "adios" => "hasta pronto"
                        ^cmd4.sc:4: unreachable code
        case "hola" => "saludos"
                       ^




    [36mstringExample[39m: [32mString[39m = [32m"hola"[39m
    [36mres4_1[39m: [32mString[39m = [32m"???"[39m



En este ejemplo, pusimos en primer caso el comodín `_`, por lo que cualquier valor al ser comparado ejecutará el código de la derecha. El resto de casos son inaccesibles, lo que hará que nos muestre un error de compilación `patterns after a variable pattern cannot match` que nos indica que tenemos código inalcanzable. Todos los casos que hay tras el `case _` se podrían borrar. Esto nos da una pista que nuestro pattern matching podría estar mal o que podemos prescindir de casos inalcanzables.

## Pero esto es solo la punta del iceberg.

En pattern matching no solo podemos hacer condiciones de igualdad, como hemos visto, si no que podemos hacer una variada cantidad de acciones.
### Comparar con múltiples casos.

En caso de tener múltiples valores que pueden satisfacer un mismo caso, podemos representarlo separandolos con `|`.


```scala
val stringExample1 = "hola"
stringExample1 match {
    case "hola" | "holi" => "saludos"
    case "adios" => "hasta pronto"
    case _ => "???"
}

val stringExample2 = "holi"
stringExample2 match {
    case "hola" | "holi" => "saludos"
    case "adios" => "hasta pronto"
    case _ => "???"
}
```




    [36mstringExample1[39m: [32mString[39m = [32m"hola"[39m
    [36mres5_1[39m: [32mString[39m = [32m"saludos"[39m
    [36mstringExample2[39m: [32mString[39m = [32m"holi"[39m
    [36mres5_3[39m: [32mString[39m = [32m"saludos"[39m



## Asignación a un valor

En caso de cumplir la condición, muchas veces necesitamos recoger el valor extraido para poder procesarlo.


```scala
val stringExample = "ya estoy"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x => s"el valor $x no está contemplado"
}
```




    [36mstringExample[39m: [32mString[39m = [32m"ya estoy"[39m
    [36mres6_1[39m: [32mString[39m = [32m"el valor ya estoy no est\u00e1 contemplado"[39m



Podemos ver, que hemos cambiado nuestro comodin `_` por `x`, lo que permite que se pueda usar en la parte derecha y podemos garantizar que si llegamos a este caso, nunca contendrá los valores `"hola"` y `"adios`". Realmente el comodín no indica "en cualquier otro caso", es una asignación igual que con `x`. La diferencia es que `_` no puede ser llamada desde la parte derecha.

Aquí quiero hacer otro inciso, y es la forma que puedes nombrar al valor donde vamos a asignar el elemento sobre el que hacemos el matching, ya que scala tiene unas reglas.

Pongamos el siguiente ejemplo, en el que asignamos el valor a un nombre ya existente.


```scala
val x = "soy x"

val stringExample = "ya estoy"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x => s"el valor $x no está contemplado"
}
```




    [36mx[39m: [32mString[39m = [32m"soy x"[39m
    [36mstringExample[39m: [32mString[39m = [32m"ya estoy"[39m
    [36mres7_2[39m: [32mString[39m = [32m"el valor ya estoy no est\u00e1 contemplado"[39m



En este caso, el valor `x` de dentro del `match` impedirá en el contexto de la derecha que se pueda acceder al valor `x` externo. Esto se llama ocultamiento de valor o "variable shadowing", y puede llevar a confusión en algunos casos. Desgraciadamente, el compilador de scala, NO nos dará una advertencia en caso de que ocurra esto.

¿Y si yo quisiera comparar un caso con el contenido de un valor que tengo definido fuera del `match`? En el pattern matching se puede, pero siguiendo unas reglas, ya que, como hemos visto, un valor con el que nos gustaría comparar puede convertirse en una nueva asignación. Entonces ¿cómo podría crear un caso si es igual a mi valor externo `x`? Indicando que este nombre de valor no es para asignarlo, si no compararlo, rodeando el valor con comillas `` `x` ``:


```scala
val x = "soy x"

val stringExample = "soy x"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case `x` => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```




    [36mx[39m: [32mString[39m = [32m"soy x"[39m
    [36mstringExample[39m: [32mString[39m = [32m"soy x"[39m
    [36mres8_2[39m: [32mString[39m = [32m"es el valor que ten\u00eda en x"[39m



Y si no te fias que esto sea así, pongamos un ejemplo que no coincida.


```scala
val x = "soy x"

val stringExample2 = "no soy x"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case `x` => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```




    [36mx[39m: [32mString[39m = [32m"soy x"[39m
    [36mstringExample2[39m: [32mString[39m = [32m"no soy x"[39m
    [36mres9_2[39m: [32mString[39m = [32m"ninguno de los anteriores"[39m



Otra forma más sencilla es seguir [la guia de estilo de scala](https://docs.scala-lang.org/style/naming-conventions.html#constants-values-variable-and-methods), en la que indica que los valores constantes definidos a nivel de clase, han de empezar con mayúscula, lo que reconoce que no es un valor a asignar


```scala
val X = "soy x"

val stringExample = "soy x"
stringExample match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case X => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```




    [36mX[39m: [32mString[39m = [32m"soy x"[39m
    [36mstringExample[39m: [32mString[39m = [32m"soy x"[39m
    [36mres10_2[39m: [32mString[39m = [32m"es el valor que ten\u00eda en x"[39m




```scala
val X = "soy x"

val stringExample2 = "no soy x"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case X => s"es el valor que tenía en x"
    case _ => "ninguno de los anteriores"
}
```




    [36mX[39m: [32mString[39m = [32m"soy x"[39m
    [36mstringExample2[39m: [32mString[39m = [32m"no soy x"[39m
    [36mres11_2[39m: [32mString[39m = [32m"ninguno de los anteriores"[39m



### Refinar la condición.

Ahora que sabemos asignar el valor dentro de un caso, podemos llegar a otra de las ventajas del pattern matching y es el poder refinar la condición sin tener que ser siempre por igualdad, como hicimos hasta el momento. Pongamos el ejemplo donde quisieramos tratar los strings que comiencen por `h` de manera distinta, excepto el caso que tenemos ya, comparando con la palabra `"hola"`. Con los conocimientos que tenemos actualmente, nuestro código quedaría algo tal que así:


```scala
val stringExample2 = "habana"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x => if (x.head == 'h') "comienza por h" else "ninguno de los anteriores"
}
```




    [36mstringExample2[39m: [32mString[39m = [32m"habana"[39m
    [36mres12_1[39m: [32mString[39m = [32m"comienza por h"[39m



Pero la utilidad del pattern matching es la de aplanar todos los posibles casos y no empezar a anidar casos más complejos, por lo que podemos hacer uso de la asignación del valor y hacer filtrado de casos de la siguiente manera:


```scala
val stringExample2 = "habana"
stringExample2 match {
    case "hola" => "saludos"
    case "adios" => "hasta pronto"
    case x if x.head == 'h' => "comienza por h"
    case _ => "ninguno de los anteriores"
}
```




    [36mstringExample2[39m: [32mString[39m = [32m"habana"[39m
    [36mres13_1[39m: [32mString[39m = [32m"comienza por h"[39m



De esta manera, vemos más claramente los 4 posibles casos estructurados con su condición primero, y no tener que rebuscar lógica escondida a lo largo de todo el código. Lo único a tener en cuenta, es que este `if` no necesita paréntesis en la condición a diferencia de la estructura de control en scala.

### Comprobación del tipo del elemento matcheado.

Hasta ahora, ¿todo bien?. Comencemos a trabajar con más tipos, a parte de nuestro querido `String`. ¿Cómo podríamos sacar un casos distintos si es string, si es Integer, y un último para el resto? En principio sería algo tal que así:



```scala
def matchfun(x: Any): String =
 x match {
     case x if x.isInstanceOf[String] =>
       val xString = x.asInstanceOf[String]
       s"tengo el string $xString"
     case x if x.isInstanceOf[Int] =>
       val xInt = x.asInstanceOf[Int]
       s"tengo el integer $xInt"
     case _ => "es otro tipo"
 }
```




    defined [32mfunction[39m [36mmatchfun[39m




```scala
matchfun("hola")
matchfun(42)
matchfun(12.4)
```




    [36mres15_0[39m: [32mString[39m = [32m"tengo el string hola"[39m
    [36mres15_1[39m: [32mString[39m = [32m"tengo el integer 42"[39m
    [36mres15_2[39m: [32mString[39m = [32m"es otro tipo"[39m



Esta forma genera mucho código repetitivo, por lo que la syntaxis del pattern matching nos da una forma más concisa de describirlo, no solo para saber si es de un tipo, también hará automáticamente el cambio de tipo para que trabajemos en nuestro contexto de la derecha.


```scala
def matchfun2(x: Any): String =
 x match {
     case x: String => s"tengo el string $x"
     case x: Int => s"tengo el integer $x"
     case _ => "es otro tipo"
 }
```




    defined [32mfunction[39m [36mmatchfun2[39m




```scala
matchfun2("hola")
matchfun2(42)
matchfun2(12.4)
```




    [36mres17_0[39m: [32mString[39m = [32m"tengo el string hola"[39m
    [36mres17_1[39m: [32mString[39m = [32m"tengo el integer 42"[39m
    [36mres17_2[39m: [32mString[39m = [32m"es otro tipo"[39m



Esta forma es mucho más concisa y nos evita mancharnos las manos con funciones como `asInstanceOf`.

#### Cuidado con las comprobaciones de algunas clases

Una cosa que tenemos que tener en cuenta, es que estas comprobaciones se hacen en tiempo de ejecución, y la JVM tiene una limitación, y es los tipos paramétricos no existen durante la ejecución. En otras palabras, que si quisieramos comparar y saber si es de tipo `List[String]` o `List[Int]` no podríamos fácilmente, ya que la información de que tipo de elemento contiene se pierde durante la ejecución:


```scala
def matchfun2(x: List[Any]): String =
 x match {
     case x: List[String] => s"tengo una lista de string de longitud ${x.size}"
     case x: List[Int] => s"tengo una lista de integer de longitud ${x.size}"
     case _ => "es otro tipo de lista"
 }
```

    cmd18.sc:3: non-variable type argument String in type pattern List[String] (the underlying of List[String]) is unchecked since it is eliminated by erasure
         case x: List[String] => s"tengo una lista de string de longitud ${x.size}"
                 ^cmd18.sc:4: non-variable type argument Int in type pattern List[Int] (the underlying of List[Int]) is unchecked since it is eliminated by erasure
         case x: List[Int] => s"tengo una lista de integer de longitud ${x.size}"
                 ^cmd18.sc:4: unreachable code
         case x: List[Int] => s"tengo una lista de integer de longitud ${x.size}"
                              ^




    defined [32mfunction[39m [36mmatchfun2[39m




```scala
matchfun2(List("hola", "adios"))
matchfun2(List(42))
matchfun2(List(4.2f))
```




    [36mres19_0[39m: [32mString[39m = [32m"tengo una lista de string de longitud 2"[39m
    [36mres19_1[39m: [32mString[39m = [32m"tengo una lista de string de longitud 1"[39m
    [36mres19_2[39m: [32mString[39m = [32m"tengo una lista de string de longitud 1"[39m



Como puedes comprobar, el patter matching no puede ver más allá de que es una lista, y el tipo contenido nunca lo tiene en cuenta, por lo que siempre entrará en el primer caso. Se podría comprobar que tipo de elemento contiene en la cabeza, pero siempre tendremos el problema en listas vacías, ya que nos resultará imposible poder comprobarlo. Eso si, como todo posible punto de error, el compilador nos informará para que lo tengamos en cuenta con el siguiente warning `List[String] is unchecked since it is eliminated by erasure`, o traducido, "List[String] no está chequeado porque ha sido borrado" ya que el compilador lo traducirá a `case x: List => ...` borrando el tipo de la lista. Como os podréis imaginar, esto no ocurre solo con listas, si no con todos los tipos paramétricos.

### ADT's en pattern matching

Ya hemos visto que scala permite comprobar el tipo del elemento para poder realizar una acción para cada tipo. Es por esto que quiero pararme a comentar una particularidad de la programación funcional, que por supuesto se aplica en scala, y es el uso de los Tipos Algebráicos de Datos, Algebraic Data Types en inglés o ADT para que sea más corto. Esto es una representación de los datos que se basa en el producto y suma de tipos, por ejemplo un producto de String e integer sería la siguiente `case class`


```scala
case class Usuario(nombre: String, edad: Int)
```




    defined [32mclass[39m [36mUsuario[39m



Y una  suma de tipos se puede representar de múltiples maneras, pero la más común es el uso de `sealed trait` por ejemplo.


```scala
sealed trait Trabajador
case class Currito(nombre: String) extends Trabajador
case class Jefe(nombre: String, subordinados: List[Trabajador]) extends Trabajador
```




    defined [32mtrait[39m [36mTrabajador[39m
    defined [32mclass[39m [36mCurrito[39m
    defined [32mclass[39m [36mJefe[39m



En este vemos una representación de lo que sería un trabajador: o es alguien con subordinados a su cargo, o es un currante sin nadie a su cargo. Al ser un `sealed trait` solo permite estas dos posibilidiades de tipo de trabajador, y no se puede extender en ningún otro lado. Esta forma de representación de datos es muy usada en scala, incluso en elementos que nos da el lenguaje, como sería Option, que tiene dos posibles elementos: Some, que indica que contiene un elemento, o None, que no contiene ninguno.

Dada la particularidad de esta suma de tipos, el pattern matching suele ser una herramienta muy común y útil para poder actuar según el tipo de dato que podamos encontrarnos.


```scala
def quienEs(t: Trabajador): String =
t match {
    case j: Jefe => s"${j.nombre} es jefe de ${j.subordinados.size} empleados"
    case c: Currito => s"${c.nombre} es un gran trabajador"
}

def tengoDato(o: Option[Int]): String =
o match {
    case s: Some[Int] => s"tenemos el valor ${s.get}"
    case None => "no tenemos valor" // en este caso, no comparamos por tipo, si no contra el objeto único que representa un Option vacío
}
```




    defined [32mfunction[39m [36mquienEs[39m
    defined [32mfunction[39m [36mtengoDato[39m




```scala
quienEs(Jefe("JM", List(Currito("Ar"), Currito("J"))))
quienEs(Currito("Ar"))

tengoDato(Some(23))
tengoDato(None)
```




    [36mres23_0[39m: [32mString[39m = [32m"JM es jefe de 2 empleados"[39m
    [36mres23_1[39m: [32mString[39m = [32m"Ar es un gran trabajador"[39m
    [36mres23_2[39m: [32mString[39m = [32m"tenemos el valor 23"[39m
    [36mres23_3[39m: [32mString[39m = [32m"no tenemos valor"[39m



### Extractores

Pero no nos quedemos solo con las limitaciones, porque al fin llega uno de los elementos más potentes del pattern matching, y mi favorito, los extractores. Pongamos que ya somos mayorcitos y no trabajamos solo con tipos simples como String, Int, etc, si no que ya tenemos estructuras más complejas, por ejemplo una tupla, en la que nos gustaría hacer varios casos, segun el contenido de esta, como hemos hecho hasta ahora, con nuestro conocimiento actual, podríamos hacer algo tal que así:


```scala
def tuplaMatch(x: (String, Int)): String =
 x match {
     case x if x._1 == "hola" & x._2  == 10 => "hola con valor igual que 10"
     case x if x._1 == "hola" & x._2 > 10 => "hola con valor mayor que 10"
     case x if x._1 == "hola" => "hola con valor menor a 10"
     case x => s"la palabra es ${x._1} con valor ${x._2}"
 }
```




    defined [32mfunction[39m [36mtuplaMatch[39m




```scala
tuplaMatch(("hola", 10))
tuplaMatch(("hola", 42))
tuplaMatch(("hola", 2))
tuplaMatch(("adios", 42))
```




    [36mres25_0[39m: [32mString[39m = [32m"hola con valor igual que 10"[39m
    [36mres25_1[39m: [32mString[39m = [32m"hola con valor mayor que 10"[39m
    [36mres25_2[39m: [32mString[39m = [32m"hola con valor menor a 10"[39m
    [36mres25_3[39m: [32mString[39m = [32m"la palabra es adios con valor 42"[39m



El código es correcto, pero hasta el momento la ventaja del pattern matching principal es una descripción muy gráfica de la lógica en la comprobación, y aquí es donde entra el uso de los extractores. Con estos, podemos comprobar o asignar los elementos internos de una manera mucho más parecida a la representación de la construcción de la clase.

Por ejemplo, para crear una tupla, la forma en la que lo hacemos es poniendo los elementos necesarios entre paréntesis y separados por una coma, en este caso a ser una tupla de dos elementos, se podría representar tal que así:


```scala
val tupla: (String, Int) = ("texto", 1)
```




    [36mtupla[39m: ([32mString[39m, [32mInt[39m) = ([32m"texto"[39m, [32m1[39m)




```scala
def tuplaMatch2(x: (String, Int)): String =
 x match {
     case ("hola", 10) => "hola con valor igual que 10" // hacemos uso de comparación
     case ("hola", x2) if x2 > 10 => "hola con valor mayor que 10" // hacemos uso de comparación y de asignación de un elemento interno
     case ("hola", _) => "hola con valor menor a 10"
     case (x1, x2) => s"la palabra es ${x1} con valor ${x2}" // asignamos ambos elementos de la tupla
 }
```




    defined [32mfunction[39m [36mtuplaMatch2[39m




```scala
tuplaMatch(("hola", 10))
tuplaMatch(("hola", 42))
tuplaMatch(("hola", 2))
tuplaMatch(("adios", 42))
```




    [36mres28_0[39m: [32mString[39m = [32m"hola con valor igual que 10"[39m
    [36mres28_1[39m: [32mString[39m = [32m"hola con valor mayor que 10"[39m
    [36mres28_2[39m: [32mString[39m = [32m"hola con valor menor a 10"[39m
    [36mres28_3[39m: [32mString[39m = [32m"la palabra es adios con valor 42"[39m



Hay que tener en cuenta que el uso de extractores no es solo para pattern matching, también se pueden  usar en las asignaciones


```scala
val tupla: (String, Int) = ("texto", 1)
val (primero, segundo) = tupla
```




    [36mtupla[39m: ([32mString[39m, [32mInt[39m) = ([32m"texto"[39m, [32m1[39m)
    [36mprimero[39m: [32mString[39m = [32m"texto"[39m
    [36msegundo[39m: [32mInt[39m = [32m1[39m



Los extractores no solo permiten comparar por igualdad, o asignar los elementos internos, también permiten comprobar el tipo de los elementos internos.


```scala
def tuplaAnyMatch(x: (Any, Any)): String =
  x match {
      case (x: String, y: String) => s"dos strings primero: $x segundo: $y"
      case (x: String, _) => s"solo el primero es string: $x"
      case (_, x: String) => s"solo el segundo es string: $x"
      case _ => "ninguno es string"
  }
```




    defined [32mfunction[39m [36mtuplaAnyMatch[39m




```scala
tuplaAnyMatch(("hola", "adios"))
tuplaAnyMatch(("hola", 42))
tuplaAnyMatch((1, "adios"))
tuplaAnyMatch((1, 42))
```




    [36mres31_0[39m: [32mString[39m = [32m"dos strings primero: hola segundo: adios"[39m
    [36mres31_1[39m: [32mString[39m = [32m"solo el primero es string: hola"[39m
    [36mres31_2[39m: [32mString[39m = [32m"solo el segundo es string: adios"[39m
    [36mres31_3[39m: [32mString[39m = [32m"ninguno es string"[39m



Como habrás visto, los extractores son una herramienta muy potente que permite simplificar nuestro código. Esto es tan común, que cuando trabajamos con `case classes` (elemento fundamental para crear ADT's) scala nos crea extractores que siguen la misma estructura de los constructores.


```scala
sealed trait Trabajador
case class Currito(nombre: String) extends Trabajador
case class Jefe(nombre: String, subordinados: List[Trabajador]) extends Trabajador
```




    defined [32mtrait[39m [36mTrabajador[39m
    defined [32mclass[39m [36mCurrito[39m
    defined [32mclass[39m [36mJefe[39m




```scala
def dibujaJerarquia(t:Trabajador, nivel: Int = 0):Unit = {
    val blancos = "  " * nivel
    t match{
        case Currito(n) => println(s"${blancos}- $n 🧑‍🏭")
        case Jefe(n, l) =>
          println(s"${blancos}- $n 😎")
          l.foreach(dibujaJerarquia(_, nivel + 1))
    }
}
```




    defined [32mfunction[39m [36mdibujaJerarquia[39m




```scala
val empresa = Jefe("A", List(
         Jefe("B", List(Currito("C"), Currito("D"))),
         Jefe("E", List(Currito("F"), Currito("G"))),
         Currito("H")
      )
    )

dibujaJerarquia(empresa)
```

    - A 😎
      - B 😎
        - C 🧑‍🏭
        - D 🧑‍🏭
      - E 😎
        - F 🧑‍🏭
        - G 🧑‍🏭
      - H 🧑‍🏭





    [36mempresa[39m: [32mJefe[39m = [33mJefe[39m(
      [32m"A"[39m,
      [33mList[39m(
        [33mJefe[39m([32m"B"[39m, [33mList[39m([33mCurrito[39m([32m"C"[39m), [33mCurrito[39m([32m"D"[39m))),
        [33mJefe[39m([32m"E"[39m, [33mList[39m([33mCurrito[39m([32m"F"[39m), [33mCurrito[39m([32m"G"[39m))),
        [33mCurrito[39m([32m"H"[39m)
      )
    )



#### Bricomanía: crea tus propios extractores

¿Y cómo puede ser esto posible? ¿Cuándo se que algo se puede descomponer o no? Muy sencillo, tenemos que ver si existe en una clase u objeto un método llamado `unapply`, este es el truco que usa scala para poder descomponer algo. Para los tipos típicos de scala, como tuplas, listas, o toda case class que creamos, scala ya tiene preparado este método para nosotros en el objeto de compañía, pero si por la razón que sea no tiene este método, podemos crearlo nosotros.

##### Extractores básicos

Lo primero a tener en cuenta son los elementos que entran en el método, en este caso siempre tiene que ser uno y del tipo que queremos descomponer, y lo que devolverá siempre ha de ser un Option, que será del tipo extraido.

Veamos un ejemplo primero, en el que queremos comprobar si un string se puede transformar a un integer. El problema es que el método toInteger que scala nos provee, lanza excepciones, por lo que, no es seguro usarlo desde un pattern matching, pero nosotros podemos crear una clase que lo permita. 


```scala
object ValidIntString { // creamos el objeto que permitirá extraer el string si es valido
    def unapply(string: String): Option[Int] = // esperamos descomponer un string, y poder sacar un integer si es posible
    try {
        Some(string.toInt) // Si logra ejecutar sin excepciones, devolverá un some con el valor
    } catch {
        case _ : Throwable => None // en caso que no fuera integer, lanzaría excepción, por lo que devolvemos None
    }
}
```




    defined [32mobject[39m [36mValidIntString[39m



Ahora podemos usar nuestro flamante nuevo extractor


```scala
"123" match {
    case ValidIntString(n) => s"es un integer con valor $n"
    case _ => "no es integer"
}

"hola" match {
    case ValidIntString(n) => s"es un integer con valor $n"
    case _ => "no es integer"
}
```




    [36mres36_0[39m: [32mString[39m = [32m"es un integer con valor 123"[39m
    [36mres36_1[39m: [32mString[39m = [32m"no es integer"[39m



Funciona perfectamente, pero, ¿y si quisiera devolver más de un elemento, como por ejemplo hacen en la tupla que vimos antes? Tenemos que devolver una tupla simplemente. Por ejemplo, queremos devolver el valor si se podía transformar a integer y el doble de este valor en una tupla.


```scala
object ValidIntStringWithDouble { // creamos el objeto que permitirá extraer el string si es valido
    def unapply(string: String): Option[(Int, Int)] = // esperamos descomponer un string, y poder sacar una tupla de integes si es posible
    try {
        val x = string.toInt
        Some(x, x * 2) // Si logra ejecutar sin excepciones, devolverá un Some con el valor
    } catch {
        case _: Throwable => None // en caso que no fuera integer, lanzaría excepción, por lo que devolvemos None
    }
}
```




    defined [32mobject[39m [36mValidIntStringWithDouble[39m




```scala
"123" match {
    case ValidIntStringWithDouble(n, n2) => s"es un integer con valor $n y su doble $n2"
    case _ => "no es integer"
}

"hola" match {
    case ValidIntStringWithDouble(n, n2) => s"es un integer con valor $n y su doble $n2"
    case _ => "no es integer"
}
```




    [36mres38_0[39m: [32mString[39m = [32m"es un integer con valor 123 y su doble 246"[39m
    [36mres38_1[39m: [32mString[39m = [32m"no es integer"[39m



##### Extractores variádico

O si necesitamos un número indeterminado de elementos a devolver podemos hacer de la variante variádico unapplySeq, en el que devolveremos una secuencia de elementos. Cuando se hace la extracción en el match, se tiene en cuenta el número de elementos que se le pasan como argumento.


```scala
object SplitDecimals { // creamos el objeto que permitirá extraer el string si es valido
    def unapplySeq(string: String): Option[List[Int]] = // esperamos descomponer un string, y devolver un número indefinido de parámetros.
    try {
        val x = string.toFloat
        val hasDecimals = x % 1 != 0 // si es par tendrá 2 elementos la lista, si es impar solo uno
        if (hasDecimals)
          Some(List((x / 1).toInt, (x % 1 * 1000000).toInt)) // Al ser par devolvemos 2 elementos
        else
          Some(List((x / 1).toInt)) // Al ser impar devolvemos solo uno
    } catch {
        case _: Throwable => None // en caso que no fuera integer, lanzaría excepción, por lo que devolvemos None
    }
}
```




    defined [32mobject[39m [36mSplitDecimals[39m




```scala
"123.0000" match {
    case SplitDecimals(n1, n2) => s"tiene decimales: $n1 . $n2"
    case SplitDecimals(n1) => s"es entero y tenemos $n1 solo"
    case _ => "no es numerico"
}

"123.56454" match {
    case SplitDecimals(n1, n2) => s"tiene decimales: $n1 . $n2"
    case SplitDecimals(n1) => s"es entero y tenemos $n1 solo"
    case _ => "no es numerico"
}

"hola" match {
    case SplitDecimals(n1, n2) => s"tiene decimales: $n1 y $n2"
    case SplitDecimals(n1) => s"es entero y tenemos $n1 solo"
    case _ => "no es numerico"
}
```




    [36mres40_0[39m: [32mString[39m = [32m"es entero y tenemos 123 solo"[39m
    [36mres40_1[39m: [32mString[39m = [32m"tiene decimales: 123 . 564537"[39m
    [36mres40_2[39m: [32mString[39m = [32m"no es numerico"[39m



#### Extractores Booleanos

Por último, scala permite otro tipo de extractor en el que no interesa extraer un elemento, si no ver si cumple una propiedad. Al igual que hacemos en la parte de refinado, en el que podemos comprobar si cumple una condición, declarándolo explicitamente. Tambien podríamos encapsular esa lógica para darle un nombre legible. Para hacer esto, también tenemos que crear un extractor con el método unapply, pero en vez de devolver un `Option`, solo tenemos que devolver un Booleano


```scala
object IsEaven {
    def unapply(int: Int): Boolean = int % 2 == 0
}
```




    defined [32mobject[39m [36mIsEaven[39m




```scala
54 match {
    case IsEaven() => "el valor es par"
    case _ => "el valor es impar"
}

45 match {
    case IsEaven() => "el valor es par"
    case _ => "el valor es impar"
}
```




    [36mres42_0[39m: [32mString[39m = [32m"el valor es par"[39m
    [36mres42_1[39m: [32mString[39m = [32m"el valor es impar"[39m



Y aunque en ejemplos anteriores siempre usamos `object`para crear nuestro extractor, también podemos hacer un extractor que requiera parámetros usando `class`. Por requermientos de la sintaxis tenemos que instanciarlo antes, para no confundir los parámetros del extractor con las comparaciones que queremos hacer.


```scala
case class GreaterThan(c: Int) {
    def unapply(int: Int): Boolean = int > c
}
```




    defined [32mclass[39m [36mGreaterThan[39m




```scala
val mayor45 = GreaterThan(45)

54 match {
    case mayor45() => "el valor es mayor que 45"
    case _ => "el valor es menor o igual"
}

23 match {
    case mayor45() => "el valor es mayor que 45"
    case _ => "el valor es menor o igual"
}
```




    [36mmayor45[39m: [32mGreaterThan[39m = [33mGreaterThan[39m([32m45[39m)
    [36mres44_1[39m: [32mString[39m = [32m"el valor es mayor que 45"[39m
    [36mres44_2[39m: [32mString[39m = [32m"el valor es menor o igual"[39m



#### Usos de extractores ya implementados en scala

Con estos ejemplos, podemos ver que los extractores no solo sirven para facilitarnos acceder a los elementos, si no que también nos permiten hacer validaciones. Por ejemplo, en scala se usa para permitir el uso de regex en pattern matching y poder extraer los elementos (o grupos) que capturamos.


```scala
val fecha = raw"(\d{4})-(\d{2})-(\d{2})".r

"2004-01-20" match {
    case fecha(year, month, day) => s"año: $year, mes: $month, dia: $day"
    case _ => "no es una fecha"
}

"hola" match {
    case fecha(year, month, day) => s"año: $year, mes: $month, dia: $day"
    case _ => "no es una fecha"
}
```




    [36mfecha[39m: [32mscala[39m.[32mutil[39m.[32mmatching[39m.[32mRegex[39m = (\d{4})-(\d{2})-(\d{2})
    [36mres45_1[39m: [32mString[39m = [32m"a\u00f1o: 2004, mes: 01, dia: 20"[39m
    [36mres45_2[39m: [32mString[39m = [32m"no es una fecha"[39m



Otro caso es poder matchera las listas, esperando un número especifico de elementos


```scala
List(1, 2, 3) match {
    case List(n1) => s"tiene solo un elemento $n1"
    case List(n1, n2) => s"tiene dos elementos $n1, $n2"
    case List(n1, n2, n3) => s"tiene tres elementos $n1, $n2, $n3"
    case l => s"tiene demasiados elementos, exactamente  ${l.size}"
}

List(1) match {
    case List(n1) => s"tiene solo un elemento $n1"
    case List(n1, n2) => s"tiene dos elementos $n1, $n2"
    case List(n1, n2, n3) => s"tiene tres elementos $n1, $n2, $n3"
    case l => s"tiene demasiados elementos, exactamente  ${l.size}"
}

List(1, 2, 3, 4) match {
    case List(n1) => s"tiene solo un elemento $n1"
    case List(n1, n2) => s"tiene dos elementos $n1, $n2"
    case List(n1, n2, n3) => s"tiene tres elementos $n1, $n2, $n3"
    case l => s"tiene demasiados elementos, exactamente  ${l.size}"
}
```




    [36mres46_0[39m: [32mString[39m = [32m"tiene tres elementos 1, 2, 3"[39m
    [36mres46_1[39m: [32mString[39m = [32m"tiene solo un elemento 1"[39m
    [36mres46_2[39m: [32mString[39m = [32m"tiene demasiados elementos, exactamente  4"[39m



### Obtener el elemento original y poder aplicar un patrón.

El uso de extractores es muy común, pero podemos llegar al caso que nos interesaría poder tener el valor original en la condición además de una extracción. Para esto, podemos hacer uso del símbolo `@` con el que podemos asignar el valor original a un valor, y a continuación del símbolo, descomponerlo con un patrón.


```scala
val fecha = raw"(\d{4})-(\d{2})-(\d{2})".r

"2004-01-20" match {
  case d @ fecha(year, month, day) => s"año: $year, mes: $month, dia: $day original $d"
  case _ => "no es una fecha"
}
```




    [36mfecha[39m: [32mscala[39m.[32mutil[39m.[32mmatching[39m.[32mRegex[39m = (\d{4})-(\d{2})-(\d{2})
    [36mres47_1[39m: [32mString[39m = [32m"a\u00f1o: 2004, mes: 01, dia: 20 original 2004-01-20"[39m



## ¡Ahora todo a la vez!

Haciendo uso de todo lo visto hasta ahora se puede realizar comparativas complejas en muy poco código:


```scala
val fecha = raw"(\d{4})-(\d{2})-(\d{2})".r // regex para fechas con guion 2020-01-01

val anio19xx = raw"19(\d{2})".r // regex para números de 4 cifras que comienzan por 19xx y extrae el xx

val anioEspecial = "2001"

def queDiaEs(dateStr: String): String =
dateStr match {
    // comparación con un valor tras la extracción
    case fecha(`anioEspecial`, _, _) => "mi año especial :D"
    // comparación con literal tras extracción
    case fecha(_, "01", "01") => s"feliz año nuevo!"
    // asignación del valor original y comparación en la extracción
    case d @ fecha(_, "02", "29") => s"es año bisiesto $d"
    // varios posibles casos de un elemento extraido
    case fecha("1800" | "1700", _, _) => "eso es muy viejo"
    // refinamiento tras extracción
    case fecha(year, month, day) if year.reverse == (month + day) => s"la fecha es capicua $year$month$day"
    // extracción de un elemento obtenido por una extracción
    case fecha(anio19xx(year19), month, day) => s"$day del $month del año $year19"
    case fecha(year, month, day) => s"año: $year, mes: $month, dia: $day"
    case _ => "no es una fecha"
}
```




    [36mfecha[39m: [32mscala[39m.[32mutil[39m.[32mmatching[39m.[32mRegex[39m = (\d{4})-(\d{2})-(\d{2})
    [36manio19xx[39m: [32mscala[39m.[32mutil[39m.[32mmatching[39m.[32mRegex[39m = 19(\d{2})
    [36manioEspecial[39m: [32mString[39m = [32m"2001"[39m
    defined [32mfunction[39m [36mqueDiaEs[39m




```scala
queDiaEs("2001-03-01")
queDiaEs("2021-01-01")
queDiaEs("2020-02-29")
queDiaEs("1800-03-29")
queDiaEs("1700-03-29")
queDiaEs("2020-02-02")
queDiaEs("1995-02-03")
queDiaEs("2021-31-10")
queDiaEs("2021")
```




    [36mres49_0[39m: [32mString[39m = [32m"mi a\u00f1o especial :D"[39m
    [36mres49_1[39m: [32mString[39m = [32m"feliz a\u00f1o nuevo!"[39m
    [36mres49_2[39m: [32mString[39m = [32m"es a\u00f1o bisiesto 2020-02-29"[39m
    [36mres49_3[39m: [32mString[39m = [32m"eso es muy viejo"[39m
    [36mres49_4[39m: [32mString[39m = [32m"eso es muy viejo"[39m
    [36mres49_5[39m: [32mString[39m = [32m"la fecha es capicua 20200202"[39m
    [36mres49_6[39m: [32mString[39m = [32m"03 del 02 del a\u00f1o 95"[39m
    [36mres49_7[39m: [32mString[39m = [32m"a\u00f1o: 2021, mes: 31, dia: 10"[39m
    [36mres49_8[39m: [32mString[39m = [32m"no es una fecha"[39m



### Un error que todos cometemos

Ya vimos al comienzo que el compilador nos daba un mensaje de advertencia cuando teníamos casos que no eran alcanzables, pero planteo otra duda. ¿Qué ocurre si tenemos un caso que no está contemplado?


```scala
def tengoDato(o: Option[Int]): String =
o match {
    case Some(0) => s"tenemos el valor 0" // solo contemplamos Some con el valor 0
    case None => "no tenemos valor"
}
```

    cmd50.sc:2: match may not be exhaustive.
    It would fail on the following input: Some((x: Int forSome x not in 0))
    o match {
    ^




    defined [32mfunction[39m [36mtengoDato[39m



Ya vemos en este caso que solo con la definición ya nos advierte el compilador de que algo falta. Pero si aun así hacemos caso omiso, al ejecutar:


```scala
tengoDato(Some(1))
```


    scala.MatchError: Some(1) (of class scala.Some)
      ammonite.$sess.cmd50$Helper.tengoDato(cmd50.sc:2)
      ammonite.$sess.cmd51$Helper.<init>(cmd51.sc:1)
      ammonite.$sess.cmd51$.<init>(cmd51.sc:7)
      ammonite.$sess.cmd51$.<clinit>(cmd51.sc:-1)


Obtenemos un error en la ejecución de tipo `scala.MatchError`. Y ya hemos dicho que esto en scala hay que evitarlo siempre que sea posible.

Como bien sabrás, scala está muy orientado a que se programe según el paradigma funcional, lo que nos lleva a las funciones puras. Si no conocías el concepto de función pura, podemos resumirlo como que a todo elemento que entra en una función, tiene que devolver un valor, pero una excepción no es un valor.

Hay que tener en cuenta que esta comprobación de exhausividad solo funciona si trabajamos con ADT's, si realizamos este match en un tipo primitivo como son `String` o `Int`, el compilador no va a poder decirnos estas advertencias.


```scala
def noExhaustivo(o: Int): String =
o match {
    case 0 => s"tenemos el valor 0"
    case 1 => s"tenemos el valor 1"
}
```




    defined [32mfunction[39m [36mnoExhaustivo[39m




```scala
noExhaustivo(0)
noExhaustivo(1)
noExhaustivo(2)
```


    scala.MatchError: 2 (of class java.lang.Integer)
      ammonite.$sess.cmd52$Helper.noExhaustivo(cmd52.sc:2)
      ammonite.$sess.cmd53$Helper.<init>(cmd53.sc:3)
      ammonite.$sess.cmd53$.<init>(cmd53.sc:7)
      ammonite.$sess.cmd53$.<clinit>(cmd53.sc:-1)


Por lo que es siempre recomendable poner un caso por defecto (`case _ => `) que lo evite si estamos haciendo match sobre un elemento primitivo o clases que no sean ADTs.

### Un truco si eres nuevo (o no te fias ni de ti mismo)

El compilador de scala sabe que un pattern matching es un posible foco de excepciones, por lo que en muchos casos, si ve que no se contemplan todos los casos de entrada, o lo que es lo mismo, no es exhaustivo, nos dará un advertencia a nivel de warning. 

Yo te doy un consejo, es una buena practica hacer que una build falle si hay algún mensaje, solo tienes que añadir la siguiente opción en tu proyecto sbt:

```scala
scalacOptions += "-Xfatal-warnings"
```

o si eres de los que trabaja con maven, en el plugin de scala añade junto a tus opciones:

```xml
<executions>
  <execution>
    <configuration>
      <args>  
        <arg>-Xfatal-warnings</arg>
      </args>
    </configuration>
  </execution>
</executions>
```


Con esto, ya tenemos todas las herramientas necesarias para convertirnos en un ninja del patter matching, pudiendo hacer una gran y compleja lógica de una manera muy legible y mantenible, y teniendo al compilador como red de seguridad que nos supervise.

## Funciones parciales

Siempre que se ha hablado de pattern matching, hemos hecho mucho hincapié en que ha de ser una función pura, es decir, contemplar todos los casos de entrada y que den respuesta a cada uno de ellos, pero fuera del `match`, algunas veces, solo queremos contemplar una parte de los casos. Esto en scala se llaman funciones parciales y tienen un [interfaz](https://www.scala-lang.org/api/current/scala/PartialFunction.html) ya creado para este propósito y que sigue la siguiente estructura:

```scala
trait PartialFunction[-A, +B]{
    def isDefinedAt(x: A): Boolean
    def apply(v1: A): B
}
```

El método `apply`, es el método que no es necesario llamarlo de forma explicita, si no que se llama directamente solo pasándole los parámetros. En este caso, sería el equivalente a la parte derecha tras la flecha `=>` del pattern matching, y como podrás imaginar, `isDefinedAt` es el método que representa la condición. Si está definido para el valor a comprobar, devuelve verdadero y ejecutaríamos el método apply.


```scala
val isEven: PartialFunction[Int, String] = new PartialFunction[Int, String]{
    def isDefinedAt(x: Int): Boolean = x % 2 == 0
    def apply(v1: Int): String = v1+" is even"
}

```




    [36misEven[39m: [32mPartialFunction[39m[[32mInt[39m, [32mString[39m] = <function1>



Esto no quiere decir que nosotros tengamos que crear una instancia de `PartialFunction` de forma explicita, porque para eso tenemos la sintaxis que usabamos anteriormente.


```scala
val isEven: PartialFunction[Int, String] = {
  case x if x % 2 == 0 => x + " is even"
}
```




    [36misEven[39m: [32mPartialFunction[39m[[32mInt[39m, [32mString[39m] = <function1>



Con esto, vemos uno de los grande secretos del pattern matching, en el fondo, solo es azúcar sintáctico que el compilador traduce al interfaz `PartialFunction`.

¿Y qué usos podemos hacer de las funciones parciales? Pues podemos aplicarlas cuando necesitamos saber dos cosas, sobre que podemos aplicar esta función, y si es el caso, que queremos hacer con ellas. Por ejemplo, el método `collect` de las colecciones, con él, seleccionamos solo los que pasan la criba, y los transformados como se indica:


```scala
List(1, 2, 3, 4).collect(isEven)
```




    [36mres56[39m: [32mList[39m[[32mString[39m] = [33mList[39m([32m"2 is even"[39m, [32m"4 is even"[39m)



Otra de las particularidades de las funciones parciales es que se pueden combinar para contemplar más casos.


```scala
val isEven: PartialFunction[Int, String] = {
  case x if x % 2 == 0 => x+" is even"
}

val isOdd: PartialFunction[Int, String] = {
  case x if x % 2 == 1 => x+" is odd"
}

val pf: PartialFunction[Int, String] = isEven.orElse(isOdd)

List(1, 2, 3, 4).map(pf)
```




    [36misEven[39m: [32mPartialFunction[39m[[32mInt[39m, [32mString[39m] = <function1>
    [36misOdd[39m: [32mPartialFunction[39m[[32mInt[39m, [32mString[39m] = <function1>
    [36mpf[39m: [32mPartialFunction[39m[[32mInt[39m, [32mString[39m] = <function1>
    [36mres57_3[39m: [32mList[39m[[32mString[39m] = [33mList[39m([32m"1 is odd"[39m, [32m"2 is even"[39m, [32m"3 is odd"[39m, [32m"4 is even"[39m)



En este caso, somos nosotros los que tenemos que asegurar que es exhaustivo.

## Aplicación de pattern matching en lambdas

Muchas veces cuando queremos hacer un pattern matching es creando una lambda, por ejemplo en un método map.


```scala
val optval = Some(4)

optval.map(x => x match {
    case 1 => "es 1"
    case 2 => "es el número 2"
    case _ => "es otro número"
})


```




    [36moptval[39m: [32mSome[39m[[32mInt[39m] = [33mSome[39m([32m4[39m)
    [36mres58_1[39m: [32mOption[39m[[32mString[39m] = [33mSome[39m([32m"es otro n\u00famero"[39m)



Como hemos visto en el ejemplo anterior, si la lógica que queremos en esa lambda solo se compone de un pattern matching, podemos simplificar el código. Scala permite realizar un pattern matching con los parámetros pasados a la lambda cambiando el inicio `x => x match {` por las llaves que tienen los casos directamente:


```scala
val optval = Some(4)

optval.map{
    case 1 => "es 1"
    case 2 => "es el número 2"
    case _ => "es otro número"
}
```




    [36moptval[39m: [32mSome[39m[[32mInt[39m] = [33mSome[39m([32m4[39m)
    [36mres59_1[39m: [32mOption[39m[[32mString[39m] = [33mSome[39m([32m"es otro n\u00famero"[39m)



Y tranquilo, si lo que esperas ha de ser una función completa o pura, ya te avisará el compilador si es exhaustivo o no (en los casos que vimos previamente).


```scala
val foo: List[Option[Int]] = List(Some(4), None, Some(1))

foo.map{
    case Some(_) => 5
}

// tenemos advertencia de compilación y además error en la ejecución
```

    cmd60.sc:3: match may not be exhaustive.
    It would fail on the following input: None
    val res60_1 = foo.map{
                         ^


    scala.MatchError: None (of class scala.None$)
      ammonite.$sess.cmd60$Helper.$anonfun$res60_1$1(cmd60.sc:3)
      ammonite.$sess.cmd60$Helper.$anonfun$res60_1$1$adapted(cmd60.sc:3)
      scala.collection.immutable.List.map(List.scala:297)
      ammonite.$sess.cmd60$Helper.<init>(cmd60.sc:3)
      ammonite.$sess.cmd60$.<init>(cmd60.sc:7)
      ammonite.$sess.cmd60$.<clinit>(cmd60.sc:-1)


Y si ve que espera una función parcial, ahí eres tú el responsable de que esté bien creado, el compilador no puede hacer todo por tí.


```scala
val foo: List[Option[Int]] = List(Some(4), None, Some(1))

foo.collect{
    case Some(_) => 5
}

// al ser parcial, no tiene responsabilidad el compilador
```




    [36mfoo[39m: [32mList[39m[[32mOption[39m[[32mInt[39m]] = [33mList[39m([33mSome[39m([32m4[39m), [32mNone[39m, [33mSome[39m([32m1[39m))
    [36mres61_1[39m: [32mList[39m[[32mInt[39m] = [33mList[39m([32m5[39m, [32m5[39m)



## Scala 3

(Nota: esto no funciona con jupyter)

Ahora toca mirar al futuro próximo, en pocos meses de la fecha de este post, saldrá una nueva versión de scala denominada dotty o scala 3. En esta se ha reescrito el compilador y va a tener muchas novedades. Respecto al tema del pattern matching, no va a tener grandes cambios, todo lo que se ha visto para indicar la condición se mantiene tal cual. Pero si merece la pena destacar unos puntos y ya de paso los escribimos con la sintaxis nueva que nos permite scala 3, omitiendo llaves y cambiandolas por `:`.

### Match como 'función'
Un pequeño cambio respecto a la palabra `match`, sigue siendo una palabra reservada, pero ahora se puede usar como llamada a un método, es decir, usando punto respecto al valor. Eso si, tras `match` se pueden eliminar las llaves, sin necesidad de poner `:`
```scala
45.match
    case 1 => "es 1"
    case 2 => "es el número 2"
    case _ => "es otro número"

```
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/qeozSdkzTvOTseiCc0OmnA)

Esta forma trata de permitirnos cambiar la prioridad para procesar el pattern matching, permitiéndonos integrarlo fácilmente con otros elementos, por ejemplo:

```scala
if 4.match
     case 5 => true
     case _ => false
then
  "valor 5"
else
  "otro valor"
````
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/A4nNc65pSTuvrtWvSkFfaA/8)

### Extractores 'irrefutables'

Otra de las mejoras que se tiene en scala 3, es la creación de extractores. Una limitación que tienen actualmente los extractores de scala 2, es la obligación de devolver los elementos extraidos en un Option, excepto para el caso del extractor Booleano, lo que representa que esa extracción puede no ir bien. Esto impide crear extractores que sabemos que siempre irán correctamente, o como lo llaman en la documentación 'irrefutables', como por ejemplo el siguiente.

```scala
object PreviousAndNextNumber:
    def unapply(int: Int): Option[(Int, Int)] = Some((int-1, int + 1))

5 match
    case PreviousAndNextNumber(_, 6) => "valor valido"
    case _ => "valor invalido"
````
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/Z0kg5fykTh6hE6c2g8GGvQ/1)

Como se ve, en este extractor se devuelve un elemento `Option[Int]` pero nunca va a haber posibilidad de que devolvamos `None`. Esto en scala 3 se ha mejorado, permitiendo el uso de extractores que no solo devuelvan `Option`, también acepta ahora `Product`. Recordad que a este último pertenecen tuplas y todas las `case class`.

```scala
object PreviousAndNextNumber:
    def unapply(int: Int): (Int, Int) = (int-1, int + 1)1

5 match
    case PreviousAndNextNumber(_, 6) => "valor valido"
    case _ => "valor invalido"

val PreviousAndNextNumber(prev, next) = 5

```
[link para ejemplo interactivo](https://scastie.scala-lang.org/alfonsorr/IYjlD9BWT9S1z6xRipQpIA/9)

Esto permite el uso de extractores en las asignaciones, cosa que no se suele usar mucho en scala dos si no conocemos perfectamente si puede lanzar excepciones en caso de no ser válida la condición.

Y como último apunte ya que hablamos de extractores, se permitirá su uso en los for compresion, pero eso es tema para otro post ;)

## Resumen final
Como habrás visto, el pattern matching es una herramienta que permite simplificar códigos muy complejos de una manera estructurada. No es algo que sea exclusivo de scala, ni siquiera es el primero en tenerlo, y otros lenguajes ya han introducido herramientas similares, pero podrás ver que siempre va de la mano con el paradigma funcional. Una tendencia que cada vez vemos en más lenguajes.
En scala siempre ha sido una de sus caracteristicas estrella, y ha madurado mucho, tanto que en nuevas versiones solo tiene algunas mejoras para poder reutilizar algunos de sus elementos en más lugares.


```scala

```
