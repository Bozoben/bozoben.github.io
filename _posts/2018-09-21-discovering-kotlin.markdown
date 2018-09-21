---
layout: post
title:  "A la découverte de kotlin avec spring 5"
date:   2018-09-21 09:53:00 +0200
categories: 
---
Ce billet est inspiré de ce [superbe tutoriel](https://spring.io/guides/tutorials/spring-boot-kotlin/){:target="_blank"}

Je vous propose de développer une petite application où chacun peut publier son salaire, et consulter celui de son voisin.
L'application est constituée de deux pages (deux templates mustache): une page d'accueil affichant les salaires, et une page permettant de saisir de nouvelles données.

# Initialisation d'un projet
Utilisez [spring initializr](https://start.spring.io/#!language=kotlin){:target="_blank"} , ou IntelliJ pour initialiser un projet Spring, avec les options suivantes : "Web", "Mustache", "JPA", "H2".

# Alors ce projet, il ressemble à quoi ?
Une fois le projet généré, prenez le temps d'examiner le fichier `build.gradle` et de vous extasier un peu.
Il y a cependant quelques points à compléter ...

Activer kotlin jpa dans `build.gradle`: 
{% highlight gradle %}
buildscript {
  dependencies {
    classpath("org.jetbrains.kotlin:kotlin-noarg:${kotlinVersion}")
  }
}
apply plugin: 'kotlin-jpa'
{% endhighlight %}


Ajoutez dans les dépendances le starter-mustache (s'il n'y est pas ...): 
{% highlight gradle %}
dependencies {
...
    compile('org.springframework.boot:spring-boot-starter-mustache')

}
{% endhighlight %}

Et dans la classe Application, activez mustache : 
{% highlight kotlin %}
@SpringBootApplication
class KothuneApplication
    @Bean
    fun mustacheCompiler(loader: Mustache.TemplateLoader?) =
        Mustache.compiler().escapeHTML(false).withLoader(loader)

    fun main(args: Array<String>) {
        runApplication<KothuneApplication>(*args)
    }
{% endhighlight %}

# Notre modèle de données
Comme souvent, tout commence par un modèle ...
On va faire simple, nous allons déclarer une seule [data class](https://kotlinlang.org/docs/reference/data-classes.html){:target="_blank"} , dans `Model.kt`:
{% highlight kotlin %}
@Entity
data class Salaire(
        val name: String,
        val job: String,
        val company: String,
        val income: String,
        @Id @GeneratedValue val id: Long? = null,
        val addedAt: LocalDateTime = LocalDateTime.now())
{% endhighlight %}
Un des intérêts des data class kotlin est la création automatique de méthodes `hashCode(), equals(), toString()`.

Créons aussi un repository:
{% highlight kotlin %}
interface SalaireRepository : CrudRepository<SalaireRepository, Long> {
    fun findAllByOrderByName(): Iterable<Salaire>
}
{% endhighlight %}

### Un petit test unitaire
Comme vous êtes super branchés test, faisons un test ...
Avant tout, il faut passer à JUnit 5, car c'est plus cool : 
`build.gradle`:
{% highlight kotlin %}
dependencies {
  testCompile('org.springframework.boot:spring-boot-starter-test') {
    exclude module: 'junit'
  }
  testImplementation('org.junit.jupiter:junit-jupiter-api')
  testRuntimeOnly('org.junit.jupiter:junit-jupiter-engine')
}
  ...

  test {
  useJUnitPlatform()
}
{% endhighlight %}  

Et une classe de test `RepositoriesTest.kt`:
{% highlight kotlin %}
@ExtendWith(SpringExtension::class)
@DataJpaTest
class RepositoriesTests(@Autowired val entityManager: TestEntityManager,
                        @Autowired val salaireRepository: SalaireRepository) {

    @Test
    fun whenFindByIdThenReturnSalaire() {
        val salaire = Salaire("Patrice", "Jardinier", "Ville de rochefort", "50000")
        entityManager.persist(salaire)
        entityManager.flush()

        val found = salaireRepository.findById(salaire.id!!)

        assertThat(found.get()).isEqualTo(salaire)
    }
}
{% endhighlight %}

Lancez les tests, si tout va bien, c'est parfait ...

# Notre contrôleur
Ce contrôleur en fait peu : il répond aux requêtes `GET /` ,`GET /newsalaire`, et `POST /newsalaire` 
{% highlight kotlin %}
@Controller
class ThuneController (private val repository: SalaireRepository) {

    @GetMapping("/")
    fun home(model: Model): String {
        model["title"] = "Kothune, Les salaires en toute transparence"
        model["salaires"] = repository.findAll()
        return "home"
    }

    @GetMapping("/newsalaire")
    fun redirectTocreate(model: Model): String {
        model["title"] = "Ajouter vos données"
        return "newsalaire"
    }

    @PostMapping("/newsalaire")
    fun create(model: Model, @ModelAttribute salaire: Salaire): String {
        repository.save(salaire)
        return "redirect:/"
    }

}
{% endhighlight %}

## Templates mustache
On crée deux templates, dans `resources/template` :
`home.mustache` - Page d'accueil, listant les salaires et permettant d'en ajouter
{% highlight html %}
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
<h1>{{ title }}</h1>
{{#salaires}}
<ul>
    <li>{{name}} - {{job}} - {{income}} - {{company}}</li>
</ul>
{{/salaires}}
<a href="/newsalaire">Ajouter</a>
</body>
</html>
{% endhighlight %}

`newsalaire.mustache` - Formulaire de saisie d'un salaire
{% highlight html %}
<html>
<head><title>{{title}}</title></head>
<body>
<h1>{{title}}</h1>
<form method="POST" action="newsalaire">
    <input type="text" name="name" placeholder="nom"/>
    <input type="text" name="job" placeholder="votre job"/>
    <input type="text" name="company" placeholder="votre société"/>
    <input type="text" name="income" placeholder="salaire mensuel"/>
    <input type="submit"/>
</form>
</body>
</html>
{% endhighlight %}

Il est temps de lancer l'application, non ? exécutez la main classe `KothuneApplication` et rendez-vous sur http://localhost:8080 ...


## Quelques compléments

Juste pour jouer, pour initialiser des données, il est possible d'ajouter un `ApplicationRunner` : 
{% highlight kotlin %}
@SpringBootApplication
class KothuneApplication
/*    @Bean
    fun mustacheCompiler(loader: Mustache.TemplateLoader?) =
        Mustache.compiler().escapeHTML(false).withLoader(loader)*/

fun main(args: Array<String>) {
    runApplication<KothuneApplication>(*args)
}

@Component
class DataInitializer(val repository: SalaireRepository) : ApplicationRunner {

    @Throws(Exception::class)
    override fun run(args: ApplicationArguments) {
        listOf("Jean", "Simone", "Alfred").forEach {
            repository.save(Salaire(name = it, job = "Job de " + it, income = it.hashCode().toString().substring(1, 5), company = "World company"))
        }
    }
}
{% endhighlight %}

Pour rendre les deux pages un peu plus belles, pourquoi ne pas intégrer un framework CSS simple comme [bulma](https://bulma.io/){:target="_blank"} ?