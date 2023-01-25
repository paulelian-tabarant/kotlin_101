# Kotlin 101

<!-- TOC -->

* [Syntaxe de base](#syntaxe-de-base)
    * [Variable](#variable)
        * [Constante](#constante)
        * [Ré-assignable](#ré-assignable)
        * [Nullable](#nullable)
        * [Chaînes de caractère](#chaînes-de-caractère)
    * [Structures de contrôle](#structures-de-contrôle)
        * [if](#if)
        * [when](#when)
        * [Opérateurs sur variables nullables](#opérateurs-sur-variables-nullables)
    * [Itérer sur des collections d'objets](#itérer-sur-des-collections-dobjets)
        * [map](#map)
        * [flatMap](#flatmap)
        * [filter](#filter)
        * [groupBy](#groupby)
        * [reduce](#reduce)
    * [Fonctions](#fonctions)
        * [Déclaration classique](#déclaration-classique)
        * [En une ligne](#en-une-ligne)
        * [Argument par défaut](#argument-par-défaut)
        * [Argument nullable](#argument-nullable)
        * [Paramètres nommés](#paramètres-nommés)
* [Classes](#classes)
    * [Évolutions depuis Java](#évolutions-depuis-java)
    * [Déclaration complète](#déclaration-complète)
    * [Variables statiques](#variables-statiques)
    * [Héritage](#héritage)
        * [Open class](#open-class)
        * [Abstract class](#abstract-class)
    * [Mots-clés de classe](#mots-clés-de-classe)
        * [data class](#data-class)
        * [enum class](#enum-class)
        * [sealed class](#sealed-class)
        * [value class](#value-class)
        * [object](#object)
    * [Surcharge d'opérateurs](#surcharge-dopérateurs)
    * [Extension de méthode](#extension-de-méthode)
* [Interfaces](#interfaces)
    * [Déclaration](#déclaration)
* [Scope Functions](#scope-functions)
    * [with() { }](#with----)
    * [.let { }](#let--)
    * [.run { }](#run--)
    * [.apply { }](#apply--)
    * [.also { }](#also--)
* [Compléments à cette sheet](#compléments-à-cette-sheet)

<!-- TOC -->

## Syntaxe de base

### Variable

#### Constante

Si la référence pointée par la variable n'est pas amenée à être modifiée (soit dans la majorité des cas), on
utilise `val` pour déclarer la variable.

```kotlin
val message: String = "hello"
// message = "hello-world" donnerait une erreur de compilation
```

Sachant que Kotlin supporte l'inférence de type, préciser `: String` pour le typage est facultatif.

#### Ré-assignable

`var` donne la possibilité de ré-assigner une valeur après l'initialisation de la variable si nécessaire.

```kotlin
var message: String = "hello"
// message: "hello"
message = "hello, world"
// message: "hello, world"
```

#### Nullable

Faire suivre le type de variable du caractère `?` indique au compilateur que sa valeur peut être `null`. Le langage ne permettra pas d'exécuter un appel de méthode directement sur une variable nullable, pour se couvrir des `NullPointerException`.

```kotlin
val nullableString: String? = stringRepository.findBy("string_id")

nullableString.chars() // Ne compilera pas
```
Comme il n'est pas possible de faire un appel sur un objet nullable, on utilise des branchements conditionnels pour
gérer les deux cas de figure (la variable est `null`, et le cas inverse, cf.
[Opérateurs sur variables nullables](#opérateurs-sur-variables-nullables))

#### Chaînes de caractère

Kotlin ajoute l'*interpolation* des variables de type `String`. Pour insérer des valeurs dynamiques à
l'intérieur d'une chaîne de caractères en dur, utiliser `$` suivi de la variable stockant la valeur.

```kotlin
val username: String = authenticatedUserProvider.getName()
val message = "Welcome home, $username !"
```

### Structures de contrôle

#### if

La spécificité du `if` en Kotlin est que chaque embranchement peut renvoyer une valeur directement assignable à une variable. Cette syntaxe peut être utilisée en remplacement de l'opérateur ternaire (`?`..`:`) de Java, absent en Kotlin.

```kotlin
val minQuantity = productsRepository.find(productId).minQuantity
val computedQuantity = if (requestedQuantity < minQuantity) minQuantity else requestedQuantity
```

#### when

Comme un `switch / case` en Java, le `when` est utile lorsque plus de deux comportements sont souhaités
selon l'état d'une variable. Comme le `if`, chaque embranchement peut renvoyer un résultat.

```kotlin
fun getMentionBac(averageMark: Double): String =
    // Chaque embranchement renvoie une chaîne de caractères
    when (averageMark) {
        this < 10 -> "Try again !"
        this > 10 && this < 12 -> "Tout juste !"
        this > 12 && this < 14 -> "Assez bien"
        this > 14 && this < 16 -> "Bien"
        this > 16 && this < 20 -> "Très bien"
        this > 20 -> "Tu passes dans le journal"
    }
```

Dans le cas ci-dessus, `when` est décliné en fonction des valeurs de la variable `model`. Si les différents cas ne sont pas
réduits à l'état d'une seule variable, il est aussi possible d'adopter la syntaxe suivante.

```kotlin
fun calculate(val op1: Double, val op2: Double, val operator: String) =
    when {
        operator == '/' && op2 != 0.0 -> op1 / op2
        operator == '*' -> op1 * op2
        operator == '+' -> op1 + op2
        operator == '-' -> op1 - op2
        else -> throw OperationNotSupportedException(op1, op2, operator)
    }
```

#### Opérateurs sur variables nullables

L'*Elvis operator* `?:` a été introduit essentiellement pour s'abstraire des blocs conditionnels du type

```kotlin
if (variable == null) doSomethingWhenNull()
```

L'*Elvis Operator* permet d'arriver au même résultat avec la syntaxe suivante

```kotlin
variable ?: doSomethingWhenNull()
```

Les variables nullables permettent aussi de récupérer la valeur d'un champ uniquement si l'objet nullable est
non `null`, avec la syntaxe `?.`. À noter que comme pour le branchement `if`, les deux cas (`null` ou non) renvoient un
résultat assignable à une variable.

```kotlin
data class Movie(val title: String, val description: String)

class MovieRepository {
    fun find(name: String): Movie? { /* ... */ }
}

// Quelque part dans une autre classe
val movie = movieRepository.find("Three Billboards outside Ebbing, Missouri")
val description = movie?.description ?: ""
```

### Itérer sur des collections d'objets

Kotlin dispose de plusieurs fonctions utilitaires sur des collections qui permettent de s'abstraire des traditionnelles
boucles `for` dans la majorité des cas. À chaque fois, le résultat renvoie une *copie* sans modifier le contenu de la
collection d'entrée.

Chaque élément de la collection initiale est accessible directement par la dénomination `it`.

#### map

`map` transforme une collection d'entrée en appliquant la même opération à chaque élément, dans l'exemple ci-dessous, ne
conserver que le champ `label` des objets `Product`.

```kotlin
val products: List<Product> = productsRepository.findAll()
// products = [ Product { id: 1, label: "Pair of slippers" }, Product { id: 2, label: "Chicken wings" } ]
val productLabels = products.map { it.label }
// productLabels = [ String { "Pair of slippers" }, String { "Chicken wings" } ]
```

#### flatMap

Lorsqu'on doit agréger plusieurs listes entre elles, contenues dans chaque élément de la collection initiale,
l'opérateur `flatMap` devient pertinent.

```kotlin
data class CarBrand(
    val name: String,
    val models: Model
)

val carBrands: List<CarBrand> = availableBrandsRepository.findAll()
/*
carBrands = [
CarBrand { name = "Renault", models = [ Model("Mégane"), Model ("Scénic") ] },
CarBrand { name = "Fiat", models = Model("Punto") }
]
*/
val carModels = carBrands.flatMap { it.models }
// carModels = [ Model("Mégane"), Model("Scénic"), Model("Punto") ]
```

#### filter

Quand on l'applique à une collection, `filter`  ne conserver que les éléments qui vérifient l'assertion passée entre
accolades.

```kotlin
import java.util.Locale.IsoCountryCode

data class Movie(val title: String, val countryCode: IsoCountryCode)

val movies: List<Movie> = moviesRepository.findAll()
/*
movies = [
Movie { title = "The hateful Eight", countryCode = "US" },
Movie { title  = "Intouchables", countryCode = "FR" },
Movie { title  = "I, Daniel Blake", countryCode = "UK" }
]
*/
val frenchMovies = movies.filter { it.countryCode == IsoCountryCode.valueOf("FR") }
// frenchMovies = [ Movie { title  = "Intouchables", countryCode = "FR" } ]
```

#### groupBy

Quand on cherche à classer les éléments d'une liste selon la valeur d'un champ des éléments, `groupBy` génère une `Map`
avec comme clé la valeur du champ, et comme valeur les éléments ayant le champ concerné à cette valeur.

```kotlin
data class Pokemon(val name: String, var healthPoints: Int) {
    fun hit(val power: Double) = healthPoints -= power
}

val deck: List<Pokemon> = pokemonDeckProvider.getRandom()
/*
 deck = [
      Pokemon { "Charmander", 120 },
      Pokemon {"Butterfree", 100 },
      Pokemon { "Sandslash", 70 }
 ]
*/

fun hitPokemon(name: String, deck: List<Pokemon>, power: Double) {
    val deckByName = deck.groupBy { it.name }
    /*
     {
         "Charmander" -> Pokemon("Charmander", 120) 
         "Butterfree" -> Pokemon("Butterfree", 100) 
         "Sandslash" -> Pokemon("Sandslash", 70) 
     }
    */
    deckByName[name]?.hit(20.0) ?: throw PokemonNotInDeckException(name)
}

hitPokemon("Sandlash", deck, 20.0)
/*
deck = [
      Pokemon { "Charmander", 120 },
      Pokemon { "Butterfree", 100 },
      Pokemon { "Sandslash", 50 }
]
 */
```

#### reduce

`reduce` s'utilise pour agréger tous les éléments d'une liste en un seul afin d'obtenir un résultat. Les éléments sont
ajoutés à un *accumulateur* qui stocke les résultats successifs au fur et à mesure que l'opération souhaitée est
appliquée à chaque élément de la liste. Cette opération est définie en tant que lambda dans les paramètres de la
fonction `reduce`.

~~~kotlin
data class Pokemon(val name: String, val healthPoints: Int) {
    fun toString(): String = "{ $name, $healthPoints health points }"
}

fun List<Pokemon>.toString() =
    this.reduce { (pokemonsString, currentPokemon) ->
        "$pokemonsString $currentPokemon /"
    }

val pokemons = listOf(Pokemon("Groudon", 120), Pokemon("Lugia", 70))
println(pokemons.toString())
// { Groudon, 120 health points } / { Lugia, 70 health points } /
~~~

Ici, par exemple, on souhaite représenter tous les Pokémon d'une liste en une seule `String`. L'opération à appliquer,
pour chaque Pokémon, est de prendre le pokémon courant et ajouter sa représentation en `String` dans `pokemonsString`,
l'accumulateur, suivi d'un `/` qui vient marquer la séparation entre chaque Pokémon affiché dans la `String` finale.

À noter que `reduce` définit une valeur initiale par défaut à l'accumumateur (pour une `String`, `""`). Si l'on souhaite
définir une autre valeur initiale, utiliser `fold`.

Pour en savoir plus : https://kotlinlang.org/docs/collection-aggregate.html#fold-and-reduce

### Fonctions

#### Déclaration classique

La syntaxe classique de déclaration de fonction englobe le corps dans des accolades, comme en java. Le type de retour et
le typage des arguments se font respectivement après le nom de méthode et après chaque argument, séparés par `:`.

`sum` prend deux arguments de type `Double` et renvoie un `Double`.

```kotlin
fun sum(a: Double, b: Double): Double {
    return a + b
}
```

Kotlin est plus permissif sur les noms de fonction : il est aussi possible de les définir via une phrase entre
*backticks* (et donc potentiellement des espaces), là où une signature traditionnelle ne le permet pas. C'est
particulièrement utile pour l'écriture de tests explicites.

```kotlin
@Test
fun `when given two floating point numbers, should compute their sum`() {
    assertThat(sum(2.0, 5.0)).isEquelTo(7.0)
}
```

#### En une ligne

Si le corps de la méthode tient sur une seule ligne, on peut le définir directement sans accolades précédé du
caractère `=`.

```kotlin
fun sum(a: Double, b: Double): Double = a + b
```

#### Argument par défaut

Ajouter une valeur par défaut (comme une assignation de variable) permet de rendre un argument de méthode optionnel.

```kotlin
fun printWelcomeMessage(firstName: String = "stranger") {
    println("Welcome, $firstName !")
}

printWelcomeMessage("Harold")
// "Welcome, Harold !"
printWelcomeMessage()
// "Welcome, stranger !"
```

#### Argument nullable

La déclaration d'un type nullable s'applique aussi aux arguments de fonction.

```kotlin
fun printWelcomeMessage(firstName: String?) {
    println("Welcome, ${firstName ?: "stranger"} !")
}

printWelcomeMessage(null)
// "Welcome, stranger !"
```

#### Paramètres nommés

Lors d'un appel de fonction, le nom de chaque paramètre assigné pour l'appel peut être spécifié explicitement.

```kotlin
fun printWelcomeMessage(firstName: String, lastName: String, city: String) {
    println("Welcome, $firstName $lastName! What's up in $city ?")
}

printWelcomeMessage(
    lastName = "Kent",
    firstName = "Clark",
    city = "New York"
)
```

Utiliser les paramètres nommés a aussi l'avantage de pouvoir s'abstraire de l'ordre dans l'assignation des paramètres.
Dans l'exemple, `lastName` est fourni avant `firstName` contrairement à l'ordre dans la déclaration de fonction.

## Classes

### Évolutions depuis Java

Pour créer une classe `Foo` avec un champ `bar` de type `String` et un constructeur pour initialiser ce champ, en Java :

```java
class Foo {
    private final String bar;

    public Foo(String bar) {
        this.bar = bar;
    }
}
```

En Kotlin, toutes ces étapes se résument en une seule ligne.

```kotlin
class Foo(private val bar: String)
```

Le fait de déclarer des variables entre parenthèses après le nom de classe a deux effets :

- définir un attribut `bar`
- définir un constructeur sur `Foo`, avec un argument `bar` qui sera automatiquement assigné à l'attribut `bar` à la
  construction

### Déclaration complète

On peut faire suivre la déclaration minimale de classe d'un corps entouré d'accolades, pour définir des attributs
en-dehors du constructeur et des méthodes.

```kotlin
class Foo(private val bar: String, baz: Int) {
    private val incrementedBaz: Int

    init {
        incrementedBaz = baz + 1
    }

    fun getBar() = bar
}

```

Après la construction, notre classe dispose d'un attribut `bar` initialisé à la valeur de l'appel au constructeur et un
attribut `incrementedBaz` correspondant à la deuxième valeur passée au constructeur à laquelle on ajoute `1`.

Le fait de ne pas spécifier `val` ou `var` devant la variable entre parenthèses `()` permet de seulement passer la
variable au constructeur sans l'associer à un attribut de classe, et interpréter sa valeur dans le bloc `init`.

```kotlin
val foo = Foo("Hello", 2)
// foo: { bar = "Hello", incrementedBaz = 3 }
val bar = foo.getBar()
// bar: "Hello"
```

### Variables statiques

Traditionnellement en Java, on déclare les variables statiques dans le corps d'une classe à l'aide du mot-clé `static`.

```java
class Toto {
    public final static String MY_STATIC_MEMBER = "This is my static member";
}
```

On y accède ensuite en spécifiant `.MY_STATIC_MEMBER` sur le nom de la classe.

```java
String singletonValue=Toto.MY_STATIC_MEMBER;
// singletonValue: "This is my static member"
```

En kotlin, les variables et méthodes statiques se déclarent dans un bloc spécifique `companion object`. Tout ce qu'il
contient est *de facto* statique. L'accès se fait de la même manière qu'en Java.

```kotlin
class Toto {
    companion object {
        const val MY_STATIC_MEMBER = "This is my static member"
    }
}

val staticMemberValue = Toto.MY_STATIC_MEMBER
// staticMemberValue: "This is my static member"
```

### Héritage

#### Open class

Par défaut, une classe ne peut pas être étendue en Kotlin. Si l'on souhaite créer des classes filles depuis cette
classe, il faut le spécifier directement avec le mot-clé `open`.

```kotlin
import java.time.Duration
import java.time.LocalDateTime
import java.util.Currency

open class Tool(
    protected val name: String,
    protected val type: Type,
    private var stock: Int
) {
    enum class Type { SCREWDRIVER, HAMMER, DRILL }

    fun isAvailable(): Boolean = stock > 0
    fun decreaseStock() {
        stock--
    }
}

class SellableTool(
    private var price: Price,
    name: String,
    type: Type,
    stock: Int
) : Tool(name, type, stock) {

    data class Price(val amount: Double, val currency: Currency)

    fun updatePrice(amount: Double) {
        this.price = Price(amount, this.price.currency)
    }
}

class RentableTool(
    private val rentDuration: Duration,
    name: String,
    type: Type,
    stock: Int
) : Tool(name, type, stock) {

    fun shouldBeReturned(rentalDateTime: LocalDateTime, currentDateTime: LocalDateTime): Boolean {
        val dueDateTime = rentalDateTime.plus(rentDuration)
        return currentDateTime.isAfter(dueDateTime)
    }
}
```

Dans l'exemple, la classe `Tool` est étendue en deux sous-classes `SellableTool` et `RentableTool` avec des logiques
spécifiques. Une partie du comportement (infos de base sur l'outil, gestion des stocks) est mutualisée. Pour garantir le
comportement de base, on doit faire appel au constructeur de la classe mère `Tool` au moment de la construction des
classes filles, d'où les deux lignes `Tool(name, type, stock)`.

#### Abstract class

Une `abstract class` n'est destinée qu'à créer des classes filles, et ne pourra pas être instanciée directement. On
l'utilise quand les comportements mutualisés ne sont pas autoportants sans spécification via l'héritage.

Un membre non défini dans la classe mère (cf. `logPrefix` ci-dessous) doit être déclaré `abstract`.

```kotlin
abstract class Logger {
    protected abstract val logPrefix: String

    fun log(message: String) {
        println("$logPrefix: $message")
    }
}

class DebugLogger : Logger() {
    override val logPrefix = "DEBUG"
}

// Ailleurs dans l'application
val debugLogger = DebugLogger()
debugLogger.log("my first log message")
// "DEBUG: my first log message"
```

### Mots-clés de classe

#### data class

On utilise régulièrement des objets voués à transmettre des données entre deux composants. Pour faciliter leur
manipulation, Kotlin a mis en place le mot-clé `data` qui permet notamment de comparer automatiquement champ par champ
sur tous les champs de deux objets `data` sans redéfinir l'opérateur `equals()` comme traditionnellement en Java.

```kotlin
data class Payment(
    val transactionId: String,
    val amount: Double,
    val paymentMethod: PaymentMethod
)

// elsewhere in the code
val firstPayment = Payment(
    transactionId = "19537593",
    amount = 1249.99,
    paymentMethod = PaymentMethod.CB
)

val secondPayment = Payment(
    transactionId = "19537593",
    amount = 1249.99,
    paymentMethod = PaymentMethod.CB
)

val arePaymentsEqual = firstPayment == secondPayment
// arePaymentsEqual: true
```

Sans le mot-clé `data`, la comparaison serait faite sur les références de `firstPayment` et `secondPayment`. Le résultat
de la comparaison donnerait `false`.

`data` ne se limite pas à la comparaison d'objets. Avec une `data class` on peut aussi par exemple déstructurer un objet
pour récupérer ses champs dans des variables distinctes.

```kotlin
val (transactionId, amount, paymentMethod) = payment
// transactionId: "19537593", amount: 1249.99, paymentMethod: PaymentMethod.CB
```

Pour connaître les fonctionnalités offertes par `data class` : https://kotlinlang.org/docs/data-classes.html

#### enum class

L'`enum class` est pertinente lorsqu'on veut définir une classe avec un nombre d'instances restreint et déjà connu à la
compilation. Autrement dit, on sait déjà quels sont les objets de cette classe que l'on va manipuler, et qui seront tous
du type de la classe de base (`EurosDenomination` dans l'exemple).

```kotlin
import README.EurosDenomination.Type.BILL
import README.EurosDenomination.Type.COIN
import README.EurosDenomination.Unit.CENTS
import README.EurosDenomination.Unit.EUROS

enum class EurosDenomination(
    val value: Int,
    val unit: Unit,
    val type: Type
) {
    ONE_CENT(1, CENTS, COIN),
    TWO_CENTS(2, CENTS, COIN),
    FIVE_CENTS(5, CENTS, COIN),
    TEN_CENTS(10, CENTS, COIN),
    TWENTY_CENTS(20, CENTS, COIN),
    FIFTY_CENTS(50, CENTS, COIN),
    ONE_EURO(1, EUROS, COIN),
    TWO_EUROS(2, EUROS, COIN),
    FIVE_EUROS(5, EUROS, BILL),
    TEN_EUROS(10, EUROS, BILL),
    TWENTY_EUROS(20, EUROS, BILL),
    FIFTY_EUROS(50, EUROS, BILL),
    ONE_HUNDRED_EUROS(100, EUROS, BILL),
    TWO_HUNDRED_EUROS(200, EUROS, BILL),
    FIVE_HUNDRED_EUROS(500, EUROS, BILL);

    enum class Type { COIN, BILL; }
    enum class Unit { EUROS, CENTS; }
}
```

Pour plus d'infos : https://www.baeldung.com/kotlin/enum

#### sealed class

La `sealed class` est plus générique que l'`enum class`. Là où l'`enum class` contraint une seule et unique classe à un
certain nombre d'instances prédéfinies, la `sealed class` contraint ses classes filles à un nombre de types prédéfinis,
déclarés nécessairement dans le même package que celle-ci.

Elle est donc pertinente lorsqu'on sait préalablement l'ensemble des sous-types d'une classe donnée dans notre
application.

```kotlin
sealed class MoneyDenomination(
    abstract val amount: Int,
    abstract val Type: Type
) {
    enum class Type { COIN, BILL; }
}

enum class EuroDenomination : MoneyDenomination {
    ONE_CENT(1, COIN),
    TWO_CENTS(2, COIN),
    FIVE_CENTS(5, COIN),

    // ...
    ONE_HUNDRED_EUROS(100, BILL),
    TWO_HUNDRED_EUROS(200, BILL),
    FIVE_HUNDRED_EUROS(500, BILL);
}

enum class PesoDenomination : MoneyDenomination {
    FIVE_CENTAVOS(5, COIN),
    TEN_CENTAVOS(10, COIN),
    TWENTY_CENTAVOS(20, COIN),

    //...
    TWO_HUNDRED_PESOS(200, BILL),
    FIVE_HUNDRED_PESOS(500, BILL),
    ONE_THOUSAND_PESOS(1000, BILL),
}
```

Comme les sous-types d'une `sealed class` sont connus au moment du *build*, le compilateur Kotlin connaît déjà tous les
types possibles lorsqu'on manipule des objets de cette classe. Cette connaissance autorise l'utilisation d'un `when`
exhaustif en cas de comportement souhaité selon les sous-types d'une `sealed class`. Il n'est pas nécessaire de fournir
de cas par défaut.

```kotlin
fun displayCatchphrase(denomination: MoneyDenomination) {
    val message: String = when (denomination) {
        EuroDenomination -> "This comes from the old world."
        PesoDenomination -> "This comes from the other side of the atlantic !"
        // "else" pas nécessaire ici
    }

    println(message)
}
```

Pour plus d'infos : https://www.baeldung.com/kotlin/sealed-classes

#### value class

Pour encapsuler une valeur unique à l'intérieur d'un objet, Kotlin propose le mot-clé `value class`. Il est bien-sûr
possible de définir une `class` classique pour cet usage, mais rajouter `value` améliore significativement la
performance.

```kotlin
@JvmInline
value class TicketId(val value: String)
```

Pour plus d'infos : https://kotlinlang.org/docs/inline-classes.html

#### object

`object` est réservé aux cas où l'on souhaite un objet unique d'une classe accessible depuis n'importe quelle classe de
l'application (pattern [singleton](https://fr.wikipedia.org/wiki/Singleton_(patron_de_conception))).

```kotlin
object ServerConfiguration(val baseUrl: String = "https://cda.test.fr/api/v1")

// Dans une classe, quelque part dans le code
val productsResponse = httpClient.get("${ServerConfiguration.baseUrl}/products")
```

### Surcharge d'opérateurs

Les opérateurs du langage (`+`, `+=`, `()`...) peuvent être surchargés pour les objets d'une classe donnée.

Par exemple, l'opérateur invoke permet d'appeler une méthode en faisant simplement suivre la variable de l'objet
par `()`.

```kotlin
class RetrieveProducts(private val productRepository: ProductRepository) {
    override fun invoke(): List<Product> = productsRepository.findAll()
}

val retrieveProducts = RetrieveProducts(gtsProductRepository)
val products = retrieveProducts()
// Pas besoin d'appeler retrieveProducts.invoke() :-)
```

Pour en savoir plus : https://kotlinlang.org/docs/operator-overloading.html

### Extension de méthode

L'extension de méthodes permet de définir une méthode sur une classe dans un contexte extérieur à sa déclaration.

Par exemple, on peut ajouter une méthode `toPriceInEuros()` à la classe `Double` native en Kotlin.

```kotlin
fun Double.toPriceInEuros() = Price(amount = this, currency = Currency.EUR)

val twoEuros = 2.0.toPriceInEuros()
// twoEuros: Price { amount = 2.0, currency = Currency.EUR }
```

Cette fonctionnalité s'applique aussi à une collection d'objets d'un certain type.

```kotlin
import java.util.Currency

data class Price(private val amount: Double, private val currency: Currency)

fun List<Price>.sumAmounts() = this.sum { it.amount }

val sum = listOf(
    Price(2.0, Currency.EUR),
    Price(12.0, Currency.EUR),
    Price(24.0, Currency.EUR),
).sumAmounts()
// sum: 38.0
```

## Interfaces

Les interfaces ont pour vocation de faire du polymorphisme, autrement dit de définir un contrat générique pour interagir
de la même manière avec différents types de classe, indifféremment de leur logique interne.

### Déclaration

Si l'on souhaite par exemple s'abstraire d'une logique de formattage dans le code appelant, on définit une
interface `DateFormatter`, avec deux implémentations pour deux usages distincts (affichage pour des ressources REST et
pour du logging)

```kotlin
import java.time.LocalDateTime
import java.time.format.DateTimeFormatter
import java.time.format.DateTimeFormatter.ofPattern

interface DateFormatter {
    val formatter: DateTimeFormatter

    fun format(dateTime: LocalDateTime): String {
        return date.format(formatter)
    }
}

class RestDateFormatter : DateFormatter {
    override val formatter: DateTimeFormatter = ofPattern("EEEEE MMMMM yyyy, à HH:mm:ss")
}

class LoggingDateFormatter : DateFormatter {
    override val formatter: DateTimeFormatter = ofPattern("yyyy-MM-dd HH:mm:ss.SSSZ")
}
```

Le code appelant (ici un use-case `RetrieveFormattedProductConsumptionDate`) n'a aucun besoin de connaître l'
implémentation concrète derrière l'interface `DateFormatter`.

```kotlin

class RetrieveFormattedProductConsumptionDate(
    val productRepository: ProductRepository,
    val dateFormatter: DateFormatter
) {
    override fun invoke(val productId: ProductId): String {
        val product = productRepository.findBy(productId)
        return dateFormatter.format(product.consumptionDate)
    }
}
```

Il suffit ensuite d'injecter l'implémentation adéquate selon le contexte d'utilisation du use-case. Par exemple, lorsque
le use-case souhaite renvoyer des informations via une API REST :

```kotlin
val restDateFormatter = RestDateFormatter()
val retrieveRestFormattedConsumptionDate =
    RetrieveFormattedProductConsumptionDate(productsRepository, restDateFormatter)
val restFormattedConsumptionDate = retrieveRestFormattedConsumptionDate()
```

Pour un use-case qui utilise la date dans des logs :

```kotlin
val loggingDateFormatter = LoggingDateFormatter()
val retrieveLoggingFormattedConsumptionDate =
    RetrieveFormattedProductConsumptionDate(productsRepository, loggingDateFormatter)
val loggingFormattedDate = retrieveDatadogFormattedConsumptionDate()
```

## Scope Functions

Parmi les standards de Kotlin, on retrouve plusieurs fonctions qui ont pour seule responsabilité d'exécuter du code dans
le contexte d'un objet. Ce sont des méthodes déjà présentes nativement sur tout objet Kotlin.

Pour consulter la doc de référence à ce sujet : https://kotlinlang.org/docs/scope-functions.html

### with() { }

`with()`, avec l'objet sur lequel il s'applique passé en paramètre, permet d'exécuter un bloc de code en ayant accès à
l'objet via `this` sans forcément le spécifier explicitement.

À prescrire quand on accède à **plusieurs champs d'un objet** et qu'il est évident qu'on utilise ses champs sur une
série d'instructions. La dernière instruction du bloc fait office du retour du `with`.

Contrairement aux autres *scope functions*, `with` ne s'applique pas à l'objet avec la syntaxe `objet.function { }`. Il
restreint le scope à l'objet passé entre parenthèses du `with()`.

```kotlin
import README.Pokemon.Generation.ONE

data class Pokemon(
    val name: String,
    val healthPoints: Int,
    val description: String,
    val generation: Generation
) {
    enum class Generation(val label: String) { ONE("One"), TWO("Two"), THREE("Three") }
}

data class PokemonRestResource(
    val name: String,
    val healthPoints: String,
    val description: String,
    val generation: String
) {
    companion object {
        fun of(pokemon: Pokemon) = with(pokemon) {
            PokemonRestResource(
                name,
                healthPoints.toString(),
                description,
                generation.label
            )
        }
    }
}

val pokemon = Pokemon(
    name = "Mew",
    healthPoints = 70,
    description = "Mew is a Psychic-type Mythical and Legendary Pokémon.",
    generation = ONE
)

val pokemonRestResource = PokemonRestResource.of(pokemon)
/*
pokemonRestResource = Pokemon {
    name = "Mew",
    healthPoints = "70",
    description = "Mew is a Psychic-type Mythical and Legendary Pokémon.",
    generation = "One"
}
 */
```

Dans le scope de `with(pokemon)`, on accède aux champs de `pokemon` directement via `field` au lieu de `pokemon.field`
habituellement. Cela permet d'éviter une suite de `pokemon.` redondants dans cette fonction qui de toute manière
n'utilise que des champs de cet objet.

### .let { }

`let` donne accès à l'objet sur lequel il est appelé via `it`. La dernière instruction du `let` fait office de retour du
bloc, sans `return` explicite. La fonction prend tout son sens lorsqu'on veut se passer d'une variable intermédiaire
dans une série d'instructions.

```kotlin
import java.time.Duration
import java.time.LocalDate
import java.time.Year

data class Movie(val id: Int, val title: String, val releaseDate: LocalDate)
data class MovieRestResource(val id: Int, val title: String, val releaseDate: String)

interface MovieRepository {
    fun find(id: Int): Movie
}

// Au niveau d'un endpoint HTTP GET /movies/:id
fun getMovie(val id: Int, movieRepository: MovieRepository): MovieRestResource =
    movieRepository.find(id).let {
        MovieRestResource(it.id, it.title, it.releaseDate)
    }
```

Le paramètre `it` peut aussi être spécifié explicitement.

```kotlin
val movieRestResource = movieRepository.find(id).let { movie ->
    MovieRestResource(movie.id, movie.title, movie.releaseDate.toString())
}
```

### .run { }

`run` donne accès à l'objet sur lequel la fonction est appelée via `this`, et renvoie la dernière expression définie
entre les accolades `{}`.

```kotlin
val movieRestResource = movieRepository.find(id).run { movie ->
    MovieRestResource(id, title, releaseDate.toString())
}
```

Il ressemble beaucoup au `let`, mis à part que le `let` donne accès à l'objet via `it` et qu'il s'appelle forcément sur
un objet avec la syntaxe `objet.let { }` : `run` peut aussi être utilisé pour grouper une série d'instructions en une
seule, là où l'on ne pourrait pas le faire autrement.

```kotlin
data class Song(val artist: String, val title: String, val genre: Genre) {
    enum class Genre { ROCK, RAP, POP, FOLK }
}

interface Logger {
    fun log(message: String)
}

interface SongRepository {
    fun find(title: String): Song?
}

fun getSong(title: String, songRepository: SongRepository, logger: Logger): Song =
    songRepository.find(id) ?: run {
        logger.log("Unknown song : $title")
        throw UnknownSongException(title)
    }
```

Sans `run`, il aurait été impossible de combiner deux instructions à droite de l'*Elvis Operator* `?:`.

### .apply { }

L'objet est disponible en tant que `this` et se retourne lui-même.

```kotlin
data class Product(
    val name: String,
    val quantity: Int,
)

fun MutableList<Product>.addEssentialStuff() =
    this.apply {
        add(Product(name = "Pair of Slippers", quantity = 1))
        add(Product(name = "Beer keg", quantity = 7))
    }

val shoppingList = mutableListOf<Product>()
val updatedShoppingList = shoppingList.addEssentialStuff()
// updatedShoppingList : [
//      Product(name=Pair of Slippers, quantity=1),
//      Product(name=Beer keg, quantity=7)
// ] 

```

`apply` est particulièrement adapté lorsqu'on souhaite chainer la fonction définie en continuer à désigner le même objet
en tant que résultat de l'appel de fonction. Dans l'exemple ci-dessus, on pourrait imaginer

```kotlin
val remainingBeers = shoppingList
    .addEssentialStuff()
    .filter { it.name == "Beer keg" }
    .map { it.quantity }
    .first()
// remainingBeers: 7
```

Note : `filter` et `map` sont nativement chainables.

### .also { }

`also` donne accès à l'objet sur lequel elle est appelée via `it`, et retourne l'objet lui-même. C'est un mélange
entre `let` et `apply`. Si un contexte donné nécessite de conserver la valeur `this` du code englobant, `also` est une
bonne alternative à `apply`.

```kotlin
data class RentableBook(val isbn: Long, val title: String, val quantity: Int, val comments: List<String>) {
    fun increaseQuantity() = quantity++
    fun addComment(comment: String) {
        comments = mutableListOf(comments).add(comment)
    }
}

interface RentableBookRepository {
    fun find(isbn: Int): RentableBook
    fun update(isbn: Int, updatedBook: RentableBook)
}

fun giveBackBook(isbn: Int, comment: String, rentableBookRepository: RentableBookRepository) {
    val updatedBook = rentableBookRepository.find(isbn)
        .also {
            it.increaseQuantity()
            it.addComment(comment)
        }

    rentableBookRepository.update(updatedBook)
}
``` 

Note : on pourrait arriver au même objectif avec `apply`.

```kotlin
fun giveBackBook(isbn: Int, comment: String, rentableBookRepository: RentableBookRepository) {
    val updatedBook = rentableBookRepository.find(isbn)
        .apply {
            increaseQuantity()
            addComment(comment)
        }

    rentableBookRepository.update(updatedBook)
}
``` 

En revanche, si besoin de garder le `this` englobant (par exemple, dans le contexte d'une fonction dans une classe, pour
accéder aux champs de la classe), le `this` serait court-circuité par `apply` en s'appliquant au résultat
de `rentableBookRepository.find(isbn)`.

## Compléments à cette sheet

- Visibilité de champs / méthodes dans une classe (`private`, `protected`, public)
- Constructeur primaire / secondaire
- Utilisation des coroutines
- Gestion des exceptions, et spécificités avec `runCatching`
