# Kotlin 101

## Syntaxe de base

### Variable

Valeur constante, immutable

```kotlin
val message = "hello"
// message = ... would result in compilation error
```

Valeur ré-assignable

```kotlin
var message = "hello"
// string: hello
string = "hello, world"
// string: hello, world
```

### Fonction

#### Déclaration

Bloc

````kotlin
fun sum(a: Double, b: Double): Double {
    return a + b
}
````

One-line

````kotlin
fun sum(a: Double, b: Double): Double = a + b
````

#### Argument optionnel

```kotlin
fun printWelcomeMessage(firstName: String?) {
    firstName ?: {
        print("Welcome, $firstname !")
        return
    }

    print("Welcome, stranger !")
}
```

#### Argument par défaut

````kotlin
fun printWelcomeMessage(firstName: String = "stranger") {
    print("Welcome, $firstName !")
}
````

## Classes

### Constructors

Membres à assigner à la construction, traditionnellement en Java

```java
class Foo {
    private String bar;

    public Foo(String bar) {
        this.bar = bar;
    }
}
```

Équivalent Kotlin

```kotlin
class Foo(val bar: String)
```

### Companion object

#### Variables statiques (membres de classe non-liés à une instance)

Traditionnellement en Java

```java
class Toto {
    public final static String MY_STATIC_MEMBER = "This is my static member";
}

// elsewhere in the code

    String singletonValue = Toto.MY_STATIC_MEMBER;
// singletonValue: "This is my static member"
```

En kotlin

```kotlin
class Toto {
    companion object {
        val MY_SINGLETON = "This is my static member"
    }
}

val singletonValue = Toto.MY_SINGLETON;
// "This is my static member"
```

Même procédé pour déclarer des méthodes statiques

### Héritage

Pour pouvoir hériter d'une classe : mot-clé `open`

Un membre non défini dans la classe mère (cf. `logPrefix` ci-dessous) doit être déclaré `abstract`.

````kotlin
open class Logger(val logPrefix: String) {

    fun log(val message: String) {
        print("$logPrefix: $message")
    }
}

class DebugLogger() : Logger("Debug: ")

val debugLogger = DebugLogger()
debugLogger.log("my first log message")
// "Debug: my first log message"
````

### Operator overloading

Les opérateurs du langage (`+`, `+=`, `()`...) peuvent être surchargés pour les objets d'une classe donnée.

ex. `invoke()`: très utile pour les usecase

```kotlin
class RetrieveProducts(val productRepository: ProductRepository) {

    override fun invoke(): List<Product> = productsRepository.findAll()
}

// ...

val retrieveProducts = RetrieveProducts(gtsProductRepository)
val products = retrieveProducts() // No need to call retrieveProducts.invoke() :-)
```

Pour en savoir plus : https://kotlinlang.org/docs/operator-overloading.html

### Method extension

Définition d'une méthode de classe n'importe où en-dehors du corps de cette classe (`{}`)

```kotlin
class Product(val id: ProductId, val label: String) {}

// ... elsewhere in the code (can be another file !)

Product.toString() = "Product ${id.value}, with label $label"
```

Note: peut aussi s'appliquer aux collections (`List`...)

```kotlin
fun List<SimplePrice>.sumOfAmounts(currency: CurrencyCode) = SimplePrice(
    currency = currency,
    amount = sumOf { it.amount }
)
```

### Classes avec propriétés spécifiques

#### `data class` : équivalent aux POJO (Plain-Old Java Objects)

```kotlin
data class Payment(
    val transactionId: String,
    val amount: Double,
    val paymentMethod: PaymentMethod
)

// elsewhere in the code
val payment = Payment(
    transactionId = "19537593",
    amount = 1249.99,
    paymentMethod = PaymentMethod.CB
)

// fields access (using destructuring, only available on data classes)
val (transactionId, amount, paymentMethod) = payment
// "19537593", 1249.99, PaymentMethod.CB
```

#### Classes "sealed"

- Classe abstraites (non instanciables directement)
- Classes qui étendent une `sealed class` contraintes à se trouver soit dans le même fichier que la classe `sealed` de
  base, soit dans le même package

> Sealed classes are designed to be used when there are a very specific set of possible options for a value, and where each of these options is functionally different – just Algebraic Data Types.

Pour plus d'infos : https://www.baeldung.com/kotlin/sealed-classes

```kotlin
sealed class Order {
    // Closed for modification
    abstract val id: OrderId
    abstract val site: Site

    data class OrderWithTickets(
        // Closed for modification
        override val id: OrderId,
        override val site: Site,
        // Open for extension
        val articles: List<OrderArticle>,
        val couponCode: CouponCode? = null,
        val participantsDetails: List<OrderPassParticipant> = emptyList()
    ) : Order()

    data class OrderWithStay(
        // Closed for modification
        override val id: OrderId,
        override val site: Site,
        // Open for extension
        val stay: OrderStay,
        val hotel: Hotel
    ) : Order()
}
```

Par exemple, on utilise les `sealed class` dans le package `commons` du projet, pour pouvoir à la fois définir un
comportement logique de base (abstrait) et dont on maîtrise l'extension (les classes qui étendent de `sealed` héritent
du comportement de base et doivent être dans le même package).

#### Classes "enum"

Spécifier un ensemble fini de valeurs de champ (et éventuellement d'implémentation de méthodes) assignables à un objet

```kotlin
enum class Site(
    val timeZone: ZoneId,
    val currency: CurrencyCode,
    val hasB2CSales: Boolean,
    val hasStays: Boolean,
    val needsEmailSending: Boolean,
    val country: Country
) {
    PARC_ASTERIX(
        timeZone = ZoneId.of("Europe/Paris"),
        currency = EUR,
        hasB2CSales = true,
        hasStays = true,
        needsEmailSending = true,
        country = FRANCE
    ),
    FRANCE_MINIATURE(
        timeZone = ZoneId.of("Europe/Paris"),
        currency = EUR,
        hasB2CSales = true,
        hasStays = false,
        needsEmailSending = true,
        country = FRANCE
    )
}
```

Pour plus d'infos : https://www.baeldung.com/kotlin/enum

#### Classes "inline"

Utile pour wrapper un type primitif dans une class qui reflète la logique métier

```kotlin
value class TicketId(val value: String)
```

Avantage : beaucoup plus performant que l'usage équivalent sans le mot-clé `value`

Pour plus d'infos : https://kotlinlang.org/docs/inline-classes.html

## Singleton

Seule instance d'une classe, accessible globalement dans le code via un simple import

```kotlin
object ServerConfiguration(var baseUrl: String) {}

// anywhere else in the code
ServerConfiguration.baseUrl = "https://cda.test.fr/api/v1"

// another place
val gtsProductsResponse = httpClient.call("${ServerConfiguration.baseUrl}/products")
```

## Interfaces

Utilisées dans une codebase pour utiliser du polymorphisme : effectuer des actions sur des objets sans se soucier de
l'implémentation sous-jacente, pourvu qu'ils respectent le contrat d'interface

### Déclaration

```kotlin
interface DateFormatter {
    fun format(date: LocalDate): String
}

class FrontDateFormatter : DateFormatter {
    override fun format(date: LocalDate): String {
        val formatter = DateTimeFormatter.ofPattern("EEEEE MMMMM yyyy, à HH:mm:ss")
        return date.format(formatter)
    }
}

class DatadogDateFormatter : DateFormatter {
    override fun format(date: LocalDate): String {
        val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSSZ")
        return date.format(formatter)
    }
}
```

### Injection de dépendance

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

### Usage en contexte avec polymorphisme

```kotlin
// front-related use case
val frontDateFormatter = FrontDateFormatter()
val retrieveFrontFormattedConsumptionDate =
    RetrieveFormattedProductConsumptionDate(productsRepository, frontDateFormatter)
val consumptionDateForFront = retrieveFrontFormattedConsumptionDate()
// "
```

```kotlin
// datadog-related use case
val datadogDateFormatter = DatadogDateFormatter()
val retrieveDatadogFormattedConsumptionDate =
    RetrieveFormattedProductConsumptionDate(productsRepository, datadogDateFormatter)
val consumptionDateForDatadog = retrieveDatadogFormattedConsumptionDate()
// "
```

## Mots-clés

### Case

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

### Elvis

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

### String interpolation

Insérer des valeurs dynamiques de `String` à l'intérieur d'un template en dur

```kotlin
val username: String = authenticatedUserProvider.getName()

val message = "Coucou, $username"
// Coucou, Astérix
```

## Transformations

```kotlin
data class Product(
    val id: String,
    val variants: List<Variant>,
    val label: String
) {

    data class Variant(
        val id: String,
        val price: Double,
        val label: String
    )

    fun List<Variant>.toString() { /* ... */
    }

    fun toString() = "Product $label. Variants: { ${variants.toString()} }"
}
```

### filter

```kotlin
val products: List<Product> = productsRepository.findAll()
// Retrieve products which have at least one variant
val productsWithVariants = products.filter { !it.variants.isEmpty() }
```

### flat map / flatten

```kotlin
val products: List<Product> = productsRepository.findAll()
// Retrieve all variants inside products
val variants = products.flatMap { it.variants }
```

### map / toMap()

À la sauce Kotlin

```kotlin
val products: List<Product> = productsRepository.findAll()
// Retrieve every product label
val productLabels = products.map { it.label }
```

### zip

Fusionner 2 listes ayant le même nombre d'éléments, en rejoignant les éléments situés au même index deux à deux avec une
stratégie de fusion donnée en paramètre

```kotlin
data class BackendProduct(
    val id: String,
    val variants: List<BackendVariant>
) {
    data class BackendVariant(
        val id: String,
        val price: Double
    )
}

val backendProducts: List<BackendProduct> = backendProductsHttpClient.getAll()
val productLabels = backendProducts.map { productLabelsRepository.find(it.id) }

// Create a Product from the two lists above
val products = backendProducts.zip(productLabels) { backendProduct, productLabel ->
    Product(
        id = backendProduct.id,
        variants = backendProduct.variants,
        label = productLabel
    )
}
```

### plus

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

### minus

Retirer certains éléments d'une liste

```kotlin
val hungerGamesChallengers = listOf("Didier", "Jean-Mi", "Gisèle", "Monique", "Bernard")
val eliminatedChallengers = setOf("Jean-Mi", "Gisèle")

hungerGamesChallengers = hungerGamesChallengers.minus(eliminatedChallengers)
// ["Didier", "Monique", "Bernard"]
```

### groupBy

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

### fold / reduce

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

## Scope Functions

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
