# Kotlin 101

## Syntaxe de base

### Variable

#### Constante

Si la référence pointée par la variable n'est pas amenée à être modifiée (soit dans la majorité des cas), on utilise `val` pour déclarer la variable.

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

Faire suivre le type de variable du caractère `?` indique au compilateur que sa valeur peut être `null`. Le langage ne permettra pas d'exécuter un appel de méthode directement sur une variable nullable.

```kotlin
val nullableString: String? = null

nullableString.chars() // Ne compilera pas
```

Comme il n'est pas possible de faire un appel sur un objet nullable, on utilise des branchements conditionnels pour gérer les deux cas de figure (la variable est `null`, et le cas inverse)

```kotlin
val nullableString: String? = stringRepository.findBy("string_id")

val derivedString: String = nullableString ?: "default_string" 
```
Si `nullableString` est `null`, `derivedString` vaudra `"defaultString"`. Sinon, c'est la valeur de `nullableString` qui sera lue. C'est l'**Elvis operator** (`?:`).

### Structures de contrôle

#### Case : TODO

Exécuter différents blocs de code en fonction de la valeur d'une variable

```kotlin
class CarFactory {
    inner enum class Model {
        RENAULT,
        CITROEN,
        FORD
    }

    fun buildModel(model: Model) =
        when (model) {
            RENAULT -> Renault()
            CITROEN -> Citroen()
            FORD -> Ford()
            else -> throw UnknownModelException("model cannot be built")
        }
}
```

#### Elvis : TODO

Exécuter un bloc de code dans le cas où la valeur à gauche de l'elvis est `null`

```kotlin
// Somewhere in another file
data class Movie(val title: String, val description: String)

class MovieRepository() {
    fun find(name: String): Movie? { /* ... */
    }
}

val movie = movieRepository.find("Three Billboards outside Ebbing, Missouri")

// If no movie found, throw exception 
val description = movie?.description ?: throw MovieNotFoundException()
```

### Chaînes de caractère

#### Interpolation : TODO

Insérer des valeurs dynamiques de `String` à l'intérieur d'un template en dur

```kotlin
val username: String = authenticatedUserProvider.getName()

val message = "Coucou, $username"
// Coucou, Astérix
```

### Itérer sur des collections d'objets

#### map

À la sauce Kotlin

```kotlin
val products: List<Product> = productsRepository.findAll()
// Retrieve every product label
val productLabels = products.map { it.label }
```

#### flatMap

```kotlin
val products: List<Product> = productsRepository.findAll()
// Retrieve all variants inside products
val variants = products.flatMap { it.variants }
```

#### filter

```kotlin
val products: List<Product> = productsRepository.findAll()
// Retrieve products which have at least one variant
val productsWithVariants = products.filter { !it.variants.isEmpty() }
```

### Opérateurs sur des collections

#### plus

Aggréger deux listes avec des éléments du même type

```kotlin
val footballMailingList = listOf("liza@edf.fr", "f.barthez@edf.fr")
val pantheonMailingList = listOf(
    "jeremy.bouhi@octo.com",
    "simon.masliah@octo.com",
    "r.girard@octo.com",
    "quentin.mino@octo.com",
    "valentin.alves@octo.com"
)

val aggregatedList = footballMailingList.plus(pantheonMailingList)
// ["liza@edf.fr", "f.barthez@edf.fr", "jeremy.bouhi@octo.com", "simon.masliah@octo.com", "r.girard@octo.com", "quentin.mino@octo.com", "valentin.alves@octo.com"]
```

#### groupBy

Générer un dictionnaire / une map à partir d'une liste d'objet, en prenant comme clé celle spécifiée en paramètres (`it`
se réfère à chaque item pour pouvoir définir une clé unique)

```kotlin
data class Pokemon(val name: String, val attacks: List<Attack>, val healthPoints: Double) {
    fun hit(val power: Double) = healthPoints -= power
}

val deck: List<Pokemon> = pokemonDeckProvider.getRandom()
// Pokemon("Charmander"), Pokemon("Butterfree"), Pokemon("Sandslash")

fun hitPokemon(name: String, deck: List<Pokemon>, power: Double) {
    val deckByName = deck.groupBy { it.name }
    /*
    {
        "Charmander" -> Pokemon("Charmander", ...),
        "Butterfree" -> Pokemon("Butterfree", ...),
        "Sandslash" -> Pokemon("Sandslash", ...)
    }
    */
    deckByName[name].hit(20.0)
}

hitPokemon("Sandlash", deck, 20.0)
```

#### fold / reduce

Opérateur d'aggrégation : accumuler tous les éléments d'une liste pour aboutir à un seul et unique résultat. On spécifie
l'opération à effectuer entre les éléments de la liste (et la valeur initiale pour `fold()`, si nécessaire) en paramètre

```kotlin
data class Pokemon(val name: String, val healthPoints: Int)
data class PokemonStatistics(val maxHealth: Int, val shortestName: String)

val deck: List<Pokemon> = pokemonDeckProvider.getRandom()
// Pokemon("Charmander", 130), Pokemon("Ho-Oh", 90), Pokemon("Sandslash", 70)
val statistics = deck.fold(
    PokemonStatistics(maxHealth = 0.0, shortestName = "arbitrary_very_long_stringgggggggg")
    { currentStatistics, pokemon ->
        PokemonStatistics(
            maxHealth = if (pokemon.healthPoints > currentStatistics.maxHealth)
                pokemon.healthPoints
            else
                currentStatistics.maxHealth,
            shortestName = if (pokemon.name.length() < currentStatistics.shortestName.length())
                pokemon.name
            else
                currentStatistics.shortestName
        )
    }
)
// PokemonStatistics(maxHealth = 130, shortestName = "Ho-Oh")
```

### Fonctions

#### Déclaration classique

La syntaxe classique de déclaration de fonction englobe le corps dans des accolades, comme en java. Le type de retour et le typage des arguments se font respectivement après le nom de méthode et après chaque argument, séparés par `:`.

`sum` prend deux arguments de type `Double` et renvoie un `Double`.

```kotlin
fun sum(a: Double, b: Double): Double {
    return a + b
}
```

#### En une ligne

Si le corps de la méthode tient sur une seule ligne, on peut le définir directement sans accolades précédé du caractère `=`.

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

Utiliser les paramètres nommés a aussi l'avantage de pouvoir s'abstraire de l'ordre dans l'assignation des paramètres. Dans l'exemple, `lastName` est fourni avant `firstName` contrairement à l'ordre dans la déclaration de fonction.

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
- définir un constructeur sur `Foo`, avec un argument `bar` qui sera automatiquement assigné à l'attribut `bar` à la construction

### Déclaration complète

On peut faire suivre la déclaration minimale de classe d'un corps entouré d'accolades, pour définir des attributs en-dehors du constructeur et des méthodes.

```kotlin
class Foo(private val bar: String, baz: Int) {
  private val incrementedBaz: Int

  init {
    incrementedBaz = baz + 1
  }

  fun getBar() = bar
}

```

Après la construction, notre classe dispose d'un attribut `bar` initialisé à la valeur de l'appel au constructeur et un attribut `incrementedBaz` correspondant à la deuxième valeur passée au constructeur à laquelle on ajoute `1`.

Le fait de ne pas spécifier `val` ou `var` devant la variable entre parenthèses `()` permet de seulement passer la variable au constructeur sans l'associer à un attribut de classe, et interpréter sa valeur dans le bloc `init`.

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
String singletonValue = Toto.MY_STATIC_MEMBER;
// singletonValue: "This is my static member"
```
En kotlin, les variables et méthodes statiques se déclarent dans un bloc spécifique `companion object`. Tout ce qu'il contient est *de facto* statique. L'accès se fait de la même manière qu'en Java.

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

Pour pouvoir hériter d'une classe : mot-clé `open`

Un membre non défini dans la classe mère (cf. `logPrefix` ci-dessous) doit être déclaré `abstract`.

```kotlin
open class Logger {
    private abstract val logPrefix: String

    fun log(val message: String) {
        print("$logPrefix: $message")
    }
}

class DebugLogger: Logger {
    private override val logPrefix = "DEBUG"
}

val debugLogger = DebugLogger()
debugLogger.log("my first log message")
// "Debug: my first log message"
```

### Mots-clés de classe

#### data class

On utilise régulièrement des objets voués à transmettre des données entre deux composants. Pour faciliter leur manipulation, Kotlin a mis en place le mot-clé `data` qui permet notamment de comparer automatiquement champ par champ sur tous les champs de deux objets `data` sans redéfinir l'opérateur `equals()` comme traditionnellement en Java.

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
Sans le mot-clé `data`, la comparaison serait faite sur les références de `firstPayment` et `secondPayment`. Le résultat de la comparaison donnerait `false`. 

`data` ne se limite pas à la comparaison d'objets. Avec une `data class` on peut aussi par exemple déstructurer un objet pour récupérer ses champs dans des variables distinctes.

```kotlin
val (transactionId, amount, paymentMethod) = payment
// transactionId: "19537593", amount: 1249.99, paymentMethod: PaymentMethod.CB
```

Pour connaître les fonctionnalités offertes par `data class` : https://kotlinlang.org/docs/data-classes.html

#### enum class

L'`enum class` est pertinente lorsqu'on veut définir une classe avec un nombre d'instances restreint et déjà connu à la compilation. Autrement dit, on sait déjà quels sont les objets de cette classe que l'on va manipuler, et qui seront tous du type de la classe de base (`EurosDenomination` dans l'exemple).

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
    ONE_CENT            (1, CENTS, COIN),
    TWO_CENTS           (2, CENTS, COIN),
    FIVE_CENTS          (5, CENTS, COIN),
    TEN_CENTS          (10, CENTS, COIN),
    TWENTY_CENTS       (20, CENTS, COIN),
    FIFTY_CENTS        (50, CENTS, COIN),
    ONE_EURO            (1, EUROS, COIN),
    TWO_EUROS           (2, EUROS, COIN),
    FIVE_EUROS          (5, EUROS, BILL),
    TEN_EUROS          (10, EUROS, BILL),
    TWENTY_EUROS       (20, EUROS, BILL),
    FIFTY_EUROS        (50, EUROS, BILL),
    ONE_HUNDRED_EUROS (100, EUROS, BILL),
    TWO_HUNDRED_EUROS (200, EUROS, BILL),
    FIVE_HUNDRED_EUROS(500, EUROS, BILL);
  
    enum class Type { COIN, BILL; }
    enum class Unit { EUROS, CENTS; }
}
```

Pour plus d'infos : https://www.baeldung.com/kotlin/enum

#### sealed class

La `sealed class` est plus générique que l'`enum class`. Là où l'`enum class` contraint une seule et unique classe à un certain nombre d'instances prédéfinies, la `sealed class` contraint ses classes filles à un nombre de types prédéfinis, déclarés nécessairement dans le même package que celle-ci.

Elle est donc pertinente lorsqu'on sait préalablement l'ensemble des sous-types d'une classe donnée dans notre application.

```kotlin
sealed class MoneyDenomination(
    abstract val amount: Int,
    abstract val Type: Type
) {
  enum class Type { COIN, BILL; }
}

enum class EuroDenomination : MoneyDenomination {
    ONE_CENT            (1, COIN),
    TWO_CENTS           (2, COIN),
    FIVE_CENTS          (5, COIN),
    // ...
    ONE_HUNDRED_EUROS (100, BILL),
    TWO_HUNDRED_EUROS (200, BILL),
    FIVE_HUNDRED_EUROS(500, BILL);
}

enum class PesoDenomination : MoneyDenomination {
  FIVE_CENTAVOS       (5, COIN),
  TEN_CENTAVOS       (10, COIN),
  TWENTY_CENTAVOS    (20, COIN),
  //...
  TWO_HUNDRED_PESOS  (200, BILL),
  FIVE_HUNDRED_PESOS (500, BILL),
  ONE_THOUSAND_PESOS(1000, BILL),
}
```

Comme les sous-types d'une `sealed class` sont connus au moment du *build*, le compilateur Kotlin connaît déjà tous les types possibles lorsqu'on manipule des objets de cette classe. Cette connaissance autorise l'utilisation d'un `when` exhaustif en cas de comportement souhaité selon les sous-types d'une `sealed class`. Il n'est pas nécessaire de fournir de cas par défaut.

```kotlin
fun displayCatchphrase(denomination: MoneyDenomination) {
    val message: String  = when (denomination) {
        EuroDenomination -> "This comes from the old world."
        PesoDenomination -> "This comes from the other side of the atlantic !"
        // "else" pas nécessaire ici
    }
    
    println(message)
}
```

Pour plus d'infos : https://www.baeldung.com/kotlin/sealed-classes

#### value class

Pour encapsuler une valeur unique à l'intérieur d'un objet, Kotlin propose le mot-clé `value class`. Il est bien-sûr possible de définir une `class` classique pour cet usage, mais rajouter `value` améliore significativement la performance.

```kotlin
@JvmInline
value class TicketId(val value: String)
```

Pour plus d'infos : https://kotlinlang.org/docs/inline-classes.html

#### object

`object` est réservé aux cas où l'on souhaite un objet unique d'une classe  accessible depuis n'importe quelle classe de l'application (pattern [singleton](https://fr.wikipedia.org/wiki/Singleton_(patron_de_conception))).

```kotlin
object ServerConfiguration(val baseUrl: String = "https://cda.test.fr/api/v1")

// Dans une classe, quelque part dans le code
val productsResponse = httpClient.get("${ServerConfiguration.baseUrl}/products")
```

### Surcharge d'opérateurs

Les opérateurs du langage (`+`, `+=`, `()`...) peuvent être surchargés pour les objets d'une classe donnée.

Par exemple, l'opérateur invoke permet d'appeler une méthode en faisant simplement suivre la variable de l'objet par `()`.

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

Les interfaces ont pour vocation de faire du polymorphisme, autrement dit de définir un contrat générique pour interagir de la même manière avec différents types de classe, indifféremment de leur logique interne.

### Déclaration

Si l'on souhaite par exemple s'abstraire d'une logique de formattage dans le code appelant, on définit une interface `DateFormatter`, avec deux implémentations pour deux usages distincts (affichage pour des ressources REST et pour du logging)

```kotlin
import java.time.LocalDateTime
import java.time.format.DateTimeFormatter
import java.time.format.DateTimeFormatter.ofPattern

interface DateFormatter {
    val formatter: DateTimeFormatter
    
    fun format(dateTime: LocalDateTime) : String {
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

Le code appelant (ici un use-case `RetrieveFormattedProductConsumptionDate`) n'a aucun besoin de connaître l'implémentation concrète derrière l'interface `DateFormatter`.

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

Il suffit ensuite d'injecter l'implémentation adéquate selon le contexte d'utilisation du use-case. Par exemple, lorsque le use-case souhaite renvoyer des informations via une API REST :

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

## Scope Functions : TODO

Parmi les standards de Kotlin, on retrouve plusieurs fonctions qui ont pour seule responsabilité d'exécuter du code dans le contexte d'un objet.

### apply()

L'objet est disponible en tant que récepteur ``this`` et il se retourne lui-même.
Utilisé dans la configuration d'un objet.
```kotlin
data class Product(
    val name: String,
    val quantity: Int,
)

val myList = mutableListOf<Product>()

fun `add essential stuff to my ShoppingList`() : List<Product> {
    return myList.apply {
        add(Product(name = "Pair of Slippers", quantity = 1))
        add(Product(name = "Beer keg", quantity = 7))
    }
}

// println(`add essential stuff to my ShoppingList`().toString()) will output : 
// [Product(name=Pair of Slippers, quantity=1), Product(name=Beer keg, quantity=7)] 

```

### also()

```
TODO also()
```

### let()

```
TODO let()
```

### run()

```
TODO run()
```

### with()

```
TODO with()
```
