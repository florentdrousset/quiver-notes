## Jeudi 5 Septembre

### Voters
Symfony appelle tous les voteurs à chaque fois que la méthode `isGranted()` ou `denyAccessUnlessGranted()`est appelée. (les deux utilisent le __authorization checker__)
A custom voter needs to implement `VoterInterface` or extend `Voter`, which makes creating a voter even easier.

__Voter::supports($attribute, $subject)__ :
Quand `isGranted()` ou `denyAccessUnlessGranted()` est appelé, le premier argument est passé en tant que `$attribute`(ex: ROLE_USER, edit...). Le second (s'il y en a un) est passé en tant que `$subject` (ex: Un objet Post, null...). Si on return `true`, c'est la méthode `voteOnAttribute` qui est appellée.

__voteOnAttribute($attribute, $subject, TokenInterface $token)__ :

On écrit ensuite le code que l'on veut et on renvoie `true` si on veut autoriser l'accès, ou `false` si on veut l'interdire !

__Résumé__ :
isGranted()
-> appelle Voter::supports
-> true appelle voteOnAttribute
-> true autorise l'accès.

Pour passer en environnement de prod, il suffit de changer la valeur de APP_ENV dans notre fichier .env à prod.

## Mardi 27 Août

On continue les requêtes avancées sur [Doctrine](quiver:///notes/1D445A66-1F49-4416-B1CF-2BE3B8457824)

## Lundi 26 Août

Principe de base des évènements :
Un évènement est quelque chose qui se passe dans l'application (ex: un utilisateur s'inscrit)
On peut 'accrocher' des listeners ou des subscribers à cet évènement, qui réagiront à cet évènement (ex: envoi d'email, flashbag, etc...)

Au lieu d'appeler notre évènement avec un //$dispatcher->addListener(RegisterEvent::NAME, [$listener, 'sendMailToUser']);
on peut aussi écrire la ligne suivante dans notre services.yaml :

```yaml
    App\EventListener\RegisterListener :
        tags:
            - { name: kernel.event_listener, event: user.register, method: sendMailToUser, priority: -10 }
```

bin/console debug:event-dispatcher


Un Listener va écouter un évènement. On la déclare soit avec un ->addListener, soit en ajoutant des tags dans notre services.yaml.
Au lieu d'avoir des listeners, on peut avoir des _subscribers_.
Un _subscriber_ va fonctionner comme un _listener_, sauf qu'il sera une sous-classe de _EventSubscriberInterface_.
L'avantage des subscribers, c'est qu'on a plus à les déclarer comme services, c'est fait tout seul grâce à l'__autowiring__.

```php
class RegisterSubscriber implements EventSubscriberInterface
{
    public $session;

    public function __construct(SessionInterface $session)
    {
        $this->session = $session;
    }

    public function sendMailToUser(RegisterEvent $e){
        // Create the message
        $message = (new \Swift_Message())
            // Add subject
            ->setSubject('Here should be a subject')

            //Put the From address
            ->setFrom(['support@mailtrap.io'])
            ->setTo($e->getUser()->getEmail())
            ->setBody('yo !');

        $this->mailer->send($message);
    }

    public function displayRegistrationMessage() {
        $this->session->getFlashBag()->add('success', 'Vous êtes enregistré ! Bienvenue.');;
    }

    public static function getSubscribedEvents() {
       return [
           RegisterEvent::NAME => ['sendMailToUser', -10],
           RegisterEvent::NAME => ['displayRegistrationMessage', 3]
       ];
    }
}
```

le controller doit uniquement gérer les entrées et sorties HTTP

On peut aussi s'accrocher sur tous les éléments du noyau. On pourrait par exemple imaginer un subscriber qui réagirait à l'évènement KernelEvents::RESPONSE.
On peut trouver tous les évènements du noyau dans `vendor/symfony/http-kernel/KernelEvents.php`

On a donc :
1. Les évènements que l'on crée nous même
2. Les évènements du noyau de Symfony

Dans le dispatcher, on a uniquement accès à la liste des évènements qui ont des écouteurs. On a malheureusement pas de méthode permettant de recenser tous les évènements.
Une méthode pourrait être de faire une recherche sur toutes les classes étendant de la classe Event.

### Cours Benoît Symfony

Cas d'utilisations des event listeners :
1. Tout ce qui est complémentaire est appelé "scénario alternatif" -> envoyer un mail au client, etc... On peut "brancher" ou "débrancher" les évènements facilement. 
2. Et en cas de traitement répété, "transversal". Ex: si on fait quelque chose dans tous les controller, on pourrait plutôt créer un listener sur l'évènement Kernel::Controller.

Donc un évènement :
Soit transversal, soit traitement complémentaire.

"Make:entity et make:auth sont bien, le reste des générateurs non"
donc pas de make:crud, etc.

https://openclassrooms.com/fr/courses/2035826-debutez-lanalyse-logicielle-avec-uml/2035851-uml-c-est-quoi

### Exercice sur les requêtes Doctrine avancées

On veut faire un tableau de bord affichant des statistiques sur notre label.
Par exemple :
1. Le nombre de titres par genre
2. Liste des pays des gens qui se sont inscrit depuis un mois
3. Afficher les 10 artistes ayant participé au plus grand nombre d'évènements
4. Montant moyen dépensé par client

On va s'aider de la [doc de Doctrine](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/dql-doctrine-query-language.html)
Si les méthodes 'built-in' ne suffisent pas, on peut utiliser trois autres syntaxes :
1. DQL (Doctrine Query Language) -> DQL as a query language has SELECT, UPDATE and DELETE constructs that map to their corresponding SQL statement types. Permet de faire une partie du SQL, mais pas tout. On récupère au final un tableau d'objets.
2. [Query Builder](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/query-builder.html). Son intérêt est que l'on va construire des requêtes avec. 

```php
createQueryBuilder('u')
->select(...)
->where(...)
->andWhere(...)

etc.
//voir les exemples dans les Repository déjà créés par Symfony.
```

3. Native SQL -> requête SQL -> ResultSetMapping. On va devoir déclarer explicitement la relation entre les paramètres de l'objet et les colonnes de notre table. Dans celles-ci on récupère des tableaux PHP.

Binding parameters to your query
Doctrine supports dynamic binding of parameters to your query, similar to preparing queries. You can use both strings and numbers as placeholders, although both have a slightly different syntax. Additionally, you must make your choice: Mixing both styles is not allowed. Binding parameters can simply be achieved as follows:

```php
<?php
// $qb instanceof QueryBuilder

$qb->select('u')
   ->from('User', 'u')
   ->where('u.id = ?1')
   ->orderBy('u.name', 'ASC')
   ->setParameter(1, 100); // Sets ?1 to 100, and thus we will fetch a user with u.id = 100

```

You are not forced to enumerate your placeholders as the alternative syntax is available:

```php

<?php
// $qb instanceof QueryBuilder

$qb->select('u')
   ->from('User', 'u')
   ->where('u.id = :identifier')
   ->orderBy('u.name', 'ASC')
   ->setParameter('identifier', 100); // Sets :identifier to 100, and thus we will fetch a user with u.id = 100
```

Note that numeric placeholders start with a ? followed by a number while the named placeholders start with a : followed by a string.

Calling setParameter() automatically infers which type you are setting as value. This works for integers, arrays of strings/integers, DateTime instances and for managed entities. If you want to set a type explicitly you can call the third argument to setParameter() explicitly. It accepts either a PDO type or a DBAL Type name for conversion.

If you've got several parameters to bind to your query, you can also use setParameters() instead of setParameter() with the following syntax:

```php

<?php
// $qb instanceof QueryBuilder

// Query here...
$qb->setParameters(array(1 => 'value for ?1', 2 => 'value for ?2'));
```



## Vendredi 23 Août

Evenements
-> Kernel
-> Application

On va les mettre dans un dossier `Event` de notre `src`.
Les events sont traités par un dispatcher. 

en parallèle:
php composer.phar require swiftmailer

On crée une classe RegisterEvent :

```php
<?php


namespace App\Event;
use Symfony\Contracts\EventDispatcher\Event;

final class RegisterEvent extends Event
{
    public const NAME = 'user.register';
    private $user;
    public function __construct(User $user) {
        $this->user = $user;
    }

    public function getUser() {
        return $this->user;
    }
}
```

Quand on crée un évènement, on peut y stocker des informations via le constructeur. On a un évènement, on peut donc le distribuer.
Si on veut utiliser cet évènement. 

On a enregistré un utilisateur. On veut ensuite que cela déclenche un évènement.

```php
$e = new RegisterEvent($user);
$dispatcher->dispatch();
$dispatch->addEventListener
```



## Jeudi 22 Août

On observe les caractéristiques d'un service à l'aide de la commande :
`bin/console debug:container container`

#### Service Id
App\Hello\HelloWorld;

troiswa.hello_world :
  alias: App\Hello\HelloWorld
  
#### [Tag](https://openclassrooms.com/fr/courses/3619856-developpez-votre-site-web-avec-le-framework-symfony/3624937-les-services-utilisation-poussee)
  
#### Public
Si on a besoin d'un service non injecté dans le constructeur, on peut l'appeler grâce à cette méthode, uniquement SI le service est public :
`$this->get('App\Hello\HelloWorld);`

#### Synthetic
Dans services.yaml, on peut définir en synthetic : true. (Utilité relative)

#### [Lazy](https://symfony.com/doc/current/service_container/lazy_services.html)

Dichotomie __eager__ / __lazy__

```php
if ($this->isSubmitted() && $this->isValid())
```

Ici en PHP, comportement paresseux. Si le submitted est faux, il ne va pas évaluer le second terme de la condition. En cas de comportement vorace (eager), l'interpréteur évaluerait tous les membres de l'expression. 

Dans le cas des services, Symfony ne va charger des services dans le conteneur qu'en cas où il en aura besoin. (lazy)
(Typiquement, Doctrine a aussi un comportement lazy)

#### Shared

Quand on utilise un service, c'est toujours le même objet : il n'est jamais recréé. On peut aussi définir le shared sur "no", et dans ce cas là on instanciera un nouvel objet à chaque fois que le service est utilisé.

-----

Pour injecter des services, on utilise classiquement l'injection de dépendance à travers le `__construct`. Cependant, on peut aussi le faire par les setters.

```php
public $container

public function setContainer(ContainerInterface $c) {
  $this->container = $c;
}

...

$s->container->getParameter($blabla)
```

Cela nous permet d'avoir des dépendances optionnelles.

Dans notre fichier `services.yaml`, on peut définir des calls.
https://symfony.com/doc/current/service_container/calls.html

```yaml
    App\Hello\HelloWorld:
        arguments:
            $p: 'Hector'
            $prenom: '%p1%'
            #ici, le %p1% fait référence à notre paramètre p1

        calls:
```



__Exercice__ : si on passe un nom d'image à notre service, on veut que celui-ci nous ressorte une présentation plus riche de cette image (avec un <figure>) par exemple. Exemple twig :

```twig
{{ e.img | figure(e.description) | raw }}
```

Quand on écrira ça, cela insérera toute la structure HTML en question dans notre template. Si le texte est trop long, prend (par exemple) les 50 premiers caractères.
Donc tout le code <img src="asset(truc)"> sera remplacé par le code twig ci-dessus.

Pour cela, on va écrire une [extension Twig](https://symfony.com/doc/current/templating/twig_extension.html)

Quand on fait hériter une classe de AbstractExtension, Symfony comprend qu'elle est une extension Twig.
Il faut ensuite déclarer les filtres avec la fonction getFilters()

__Exercice__ : on a utilisé l'API Wiki

__Exercice__ :
partie symfony : 1 route + 1 controller
route /artiste/{id}/sameStyle

partie JS :
route de type /artiste/24/same-style

```js
fetch().then(function(response))...
```





## Mercredi 21 Août

#### Services
Quand Symfony démarre, l'une des premières choses qu'il fait est de construire le _conteneur de services_, qui gère toutes les dépendandes entre les différents services. 

* Exercice : construction de notre premier service

Service qui va afficher sur la page d'accueil la chaîne de caractère qui nous sera fournie par le service. Dans le dossier src/, on crée un nouveau dossier /hello. Dans ce dossier, on crée un fichier HelloWorld.php. Ce fichier va contenir une classe HelloWorld, avec une méthode 'yo' qui renvoie 'yo'.  

Dans la méthode de notre controller, on peut directement __injecter des services__ en paramètres de notre méthode.

```php
class WelcomeController extends AbstractController {
  //On passe notre service (classe) HelloWorld en argument, et on peut ensuite employer notre classe au sein de la méthode de notre controller
    public function welcome(EventRepository $eventRepository, \App\Hello\HelloWorld $h) {
       return $this->render(
           'index/indexView.html.twig', [
               'event' => $eventRepository->findAll(),
               'message' => $h->yo()
           ]
       );
    }
}
```

Un ajout de service classique est le service Request.

```php
public function index(Request $request, $v1, $v2)
```

Dans le fichier __service.yaml__, on peut définir des "constantes" de projet appelés parameters. Entre %, on utilise des variables yaml.

Pour y accéder, on utilise la méthode getParameter().

(Dans notre fichier .env, on peut aussi définir des constantes globales)

Dans notre service.yaml, on a ensuite la section 'services' qui contient la section 'default'.

L'auto-
wiring permet d'organiser la dépendance des services entre eux sans avoir à déclarer quoi que ce soit.
L'autoconfigure 

Dans notre App\, on veut considérer que toutes les classes qui sont dans le dossier src sont considérées comme des services. 

On peut obtenir une liste de tous les services de notre application avec la commande

```php
bin/console debug:container
```

On peut aussi ajouter un nom de service ou une chaîne de caratère à la fin de cette commande pour rechercher un service en particulier. S'il n'y a qu'un seul nom de service qui correspond, on obtient des détails sur le service en question.

On peut aussi associer des services à d'autres services (?) avec le système de tags.

###### arguments
-> constante
-> service (dépendance)
-> paramètre

###### calls

###### tags

On crée un constructeur dans notre service HelloWorld car on veut lui ajouter des paramètres. Problème : on n'instanciera jamais notre HelloWorld. Cest à cela que va nous service notre fichier de configuration services.yaml. On va lui ajouter une section !

```yaml
App\Hello\HelloWorld:
        arguments:
            $p: 'Hector'
```

Grâce à ceci, l'argument $p de notre service sera 'Hector'.

```php
class HelloWorld {
    private $name;

    public function __construct(string $p)
    {
       $this->name = $p;
    }

    public function yo() {
        return 'yo '.$this->name;
    }
}
```

On pourrait également utiliser nos paramètres yaml :

```yaml
    App\Hello\HelloWorld:
        arguments:
            $p: 'Hector'
            $prenom: '%p1%'
            #ici, le %p1% fait référence à notre paramètre p1
```

* Ex. 2 : On veut faire un service qui transforme une chaîne de caractères en maj, ou en minuscules... Et on veut que notre classe HelloWorld dépende de ce deuxième service.

On crée donc un deuxième service Magnifier.

```php
class Magnifier
{
    public function upper(string $s) {
        return strtoupper($s);
    }
}
```

On passe ensuite notre magnifier dans la signature du constructeur de notre service 'HelloWorld'. On peut ainsi utiliser notre magnifier au sein de notre service HelloWorld. 

```php
use App\Magnifier\Magnifier;

class HelloWorld {
    private $name;
    private $m;

    public function __construct(string $p, string $prenom, Magnifier $m)
    {
       $this->name = $prenom;
       $this->magnifier = $m;
    }

    public function yo() {
        return 'yo '.$this->name;
    }

    public function yoUpper() {
        return $this->magnifier->upper($this->yo());
    }
}
```

L'injection de dépendances est complètement gérée par le conteneur de services de Symfony. Quand vous avez des services, Symfony observe la signature de la fonction et remarque que le service a besoin d'une instance de la classe magnifier. Si Symfony observe que cette classe est un service, il comprend que l'on a besoin d'une instance de cette classe, grâce à __l'autowire__ du conteneur de services.

On peut aussi passer un nom de service de la manière suivante :

```yaml
$m: '@App\HelloWorld\NomDeService'
```



## Mardi 20 Août 2019

J'ai implémenté un système d'upload de fichier dans mes classes Event et Product. Plusieurs étapes pour cela, toutes tirées du [tutoriel de la doc Symfony](https://symfony.com/doc/current/controller/upload_file.html)
1. Rajouter une 

On regarde dans [Security.yaml](quiver:///notes/4E0DB276-8A3E-47B1-92B1-CE85F00C90FD)

```sh
make:user 
```

-> Entity
-> Repository __(classe de requête)__
-> [Security.yaml](quiver:///notes/4E0DB276-8A3E-47B1-92B1-CE85F00C90FD)

providers
users:
entity:
class:

```sh
make:entity
```

make:auth -> [Guard](https://symfony.com/doc/current/security/guard_authentication.html) -> LoginFormAuthenticator

```sh
make:registration-form
```

 permet de créer un formulaire d'inscription.

Les controllers de Symfony héritent de la classe Controller, qui elle même implémente le trait ControllerTrait. Dans celle-ci, on y trouve la méthode `$this->getUser()` qui permet de récupérer un utilisateur. Si personne n'est connecté, cela renvoie quand même un résultat. On va donc vouloir vérifier que ce qui est renvoyé est une instance de la classe User.

On a aussi une méthode `$this->isGranted('ROLE_ADMIN')`. Qui permet de savoir si l'utilisateur connecté possède tel ou tel rôle. (admin en l'occurence). Cette méthode peut aussi être remplacée par une annotation : `@IsGranted('ROLE_ADMIN')`

On a aussi une méthode `$this->denyAccessUnlessGranted('ROLE_ADMIN')`. Si la personne n'a pas le rôle Admin, on lui refuse l'accès et renvoie une erreur.

---

On peut aussi utiliser un objet `app.user` ou `is_granted` directement dans [Twig](quiver:///notes/CDFDCD5D-97CC-4FAE-9B98-3B439ECCBDFF)

```twig
 {% if is_granted('ROLE_MANAGER') == false %}
                  my message 
 {% endif %}
```

```php
$artist = $repository->find(25);
return ... ['a' => $artist];
// On aura donc accès à 'a' dans Twig.
```

```twig
{% for e in a.events %}
{# on parcourt les évènements de l'artist. En tâche de fond, il se passe $artist->getEvents. On fait appel à Doctrine qui va faire une requête SQL. Au moment où cette ligne est executée, on relance une requête via le modèle ! #}
{{ e.heureDebut }}
{% endfor %}
```

[Twig](quiver:///notes/CDFDCD5D-97CC-4FAE-9B98-3B439ECCBDFF) récupère en fait les informations minimum de 'Artist', et récupèrera les infos à la volée via Doctrine au moment ou notre boucle Twig sera lue.

----

3 niveaux de sécurité dans Symfony :
- Security.yaml
- Ce qu'on va mettre dans nos controllers
- Voteurs



## Lundi 19 Août 2019

Création d'une page à partir d'une méthode d'un contrôleur grâce au [tutoriel de Symfony](https://symfony.com/doc/current/page_creation.html)

Création d'utilisateurs, 4 étapes :
- Créer une (ou des) entité(s) utilisateur. -> Liée au noyau de Symfony
- Gestion de la connexion
- Configuration (dossier config) (voteurs -> ACL ((Access Control List))
- Appeler les utilisateurs -> gestion des droits d'accès

Pour créer une entité user 'automatiquement', on peut utiliser la commande suivante :

```sh
bin/console make:user
```

Cela nous permet d'avoir une méthode getRole, getPassword, getSalt, getUsername, eraseCredentials...

```sh
bin/console make:auth
```

En choisissant l'option login form, on pourra ensuite accéder à notre formulaire par l'URL /login

```sh
On peut aussi utiliser la commande
bin/console make:registration-form
```

pour crééer un formulaire d'enregistrement.

Les méthodes et routes login et logout se trouvent dans le SecurityController ainsi créé.

On gère les rôles dans security.yaml

bin/console make:fixtures

https://github.com/fzaninotto/Fake

```php
<?php

namespace App\DataFixtures;

use App\Entity\Artist;
use App\Entity\Event;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\Persistence\ObjectManager;
use Faker;

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        $faker = \Faker\Factory::create('fr_FR');
        // $product = new Product();
        // $manager->persist($product);
        for($i = 0 ; $i < 10; $i++)
        {
            $artist = new Artist();
            $artist->setNom($faker->name);
            $artist->setPays($faker->country);
            $artist->setStyle($faker->jobTitle);
            $artist->setPresentation($faker->catchPhrase);
            $manager->persist($artist);
        }


        for($i = 0 ; $i < 10; $i++)
        {
            $event = new Event();
            $event->setType($faker->companySuffix);
            $event->setDateDebut($faker->dateTime($max = 'now', $timezone = null));
            $event->setDateFin($faker->dateTime($max = 'now', $timezone = null));
            $event->setLieu($faker->company);
            $event->setVille($faker->city);
            $event->setDescription($faker->catchPhrase);
            $event->setPrix($faker->numberBetween($min = 5, $max = 150));
            $manager->persist($event);

        }
        

        $manager->flush();
    }
}
```

bin/console doctrine:fixtures:load --append

Le flag --append permet d'ajouter les données sans purger la base de données !



### Mardi 13 Août 2019

On installe un projet Symfony 4 avec la commande

Pour lancer le serveur :  php -S localhost:8000 -t public/

Contrairement à Symfony 3, il n'y a pas de DefaultController donc on commence sur une erreur 404.

app.php -> production
app_dev.php -> developpement

```twig
composer create-project symfony/website-skeleton nom-de-projet
```

On peut utiliser "make" dans la console pour créer différentes choses, par exemple un contrôleur :

```twig
bin/console make:controller
```

Projet maison de disques :

User stories. Acteur -> Moyen -> Action

![disque_diagramme.png](resources/1B05B3D232BB40A60F612F754F6A5543.png =784x1028)

Après avoir créé notre diagrame, on veut créer les entités symfony correspondantes.

```twig
 bin/console make:entity
```

Grâce à ça, on a créé une classe "Artiste" dans src/Entity et "ArtisteRepository" dans src/Repository.

On créée ensuite la classe "Evenement" de la même manière. On veut lier notre classe "Evenement" à notre classe "Artiste". Pour cela, on lui rajoute un champ "artiste_id" avec le type "relation". On peut ensuite choisir "manyToOne" (par exemple). à la question

 Do you want to add a new property to Artiste so that you can access/update Evenement objects from it - e.g. $artiste->getEvenements()? (yes/no) [yes]:

, si on répond oui, on peut aussi crééer une relation 1,n. Donc, requête doctrine qui permet de récupérer tous les évènements associés à un artiste.

```sh
Do you want to activate orphanRemoval on your relationship?
 A Evenement is "orphaned" when it is removed from its related Artiste.
 e.g. $artiste->removeEvenement($evenement)
 
 NOTE: If a Evenement may *change* from one Artiste to another, answer "no".

 Do you want to automatically delete orphaned App\Entity\Evenement objects (orphanRemoval)? (yes/no) [no]:

```

Dans ce cas, si on supprime un Artiste, on se retrouvera avec des évènements orphelins. Si on répond oui, doctrine supprimera automatiquement les évènements qui n'ont plus d'artistes. "No" est donc plus secure dans ce cas-là.

__Généralement, on ne crée par la relation One to Many, mais la Many to One.__

```sh
bin/console doctrine:database:create
bin/console doctrine:schema:create (ou update)
```

```sh
bin/console make:crud Artist
```

On a eu une erreur sur le fait qu'Event voulait afficher Artist en string. Pour corriger le problème, on a implémenté la méthode __toString dans la classe Artiste pour corriger le problème.

En lançant la commande _make:crud_, Symfony créée automatiquement les controllers et vues nécessaire. 

Les classes de repository héritent de classes plus générales.
->findAll() permet d'extraire toutes les données d'une table.
->find(id) permet de sélectionner par l'id.
->findBy('ville', 'bordeaux')
->findByVille('bordeaux')

Doctrine nous ramènera donc des objets.

On trouve également d'autres exemples dans nos classes Repository créées par défaut.

Dans la classe ArtistController, on a également la méthode _new_.

grâce au createForm, si on lui passe un objet $artist en deuxième paramètre, notre formulaire sera prérempli avec le contenu de cet objet.

->handleRequest gère notre requête en POST, met les bonnes valeurs dans les bons objets, etc.

On peut tester si un formulaire est submitted et s'il est valide. (classe de validation).
Si tout est valide, grâce à la méthode persist de l'Entity manager, on previent doctrine que l'on est en train de créer un objet dans le mécanisme de persistence. (car notre objet existe dans le code PHP mais n'existe pas encore dans la base de données. On doit cependant prvenir doctrine qu'il doit déà être pris en compte.)

->flush() permet de remplir la base de données avec le écontenu de notre objet.

On fait ensuite une redirection HTTP avec la méthode redirectToRoute. 

Pour créer la view qui correspond à notre formulaire, on passe à l'index 'array' la méthode $form->createView().

_Rappel_ : Dans Symfony, on ne manipule jamais les URL en temps que tel mais toujours les noms de route.

#### Formulaires

Grâce au make:crud, Symfony a aussi créé des formulaires dans src/Form. 

La méthode principale des formulaires est la méthode ->buildForm(). On ajoute des propriétés du $builder avec la méthode ->add().

Ne pas hésiter à lire la doc sur les [Form Types](https://symfony.com/doc/current/reference/forms/types/). Symfony est capable de comprender le type de données qu'on a défini en créant nos entités et créer le bon type de champ pour chaque champ du formulaire.

```php
->add('artist_id', EntityType::class, [
                'class' => Artist::class,
                'choice_label' => 'name'
            ])
```

Grâce à ca, on a plus a ajouter le toString dans notre controller Artist.aa---





### Lundi 12 Août 2019

Pour installer Symfony avec composer :

```php
composer create-project symfony/framework-standard-edition
```

Déjà prêt à l'emploi, version "old school" avec des ressources en démonstration

```php
symfony/website-skeleton
```

Un peu similaire à la première, mais pas d'exemples en particulier.

```php
symfony/skeleton
```

Version "flex" qui n'installe que le noyau minimum du framework.

Nous sommes donc sur "symfony/symfony": "3.4.30". La page d'accueil se situant dans web/

(Site semver.org => bonne pratiques pour nommer les versions d'un logiciel.)

Versions standard et LTS(Long-Term Support). Pour symfony 2 : 2.7/2.8 Pour Symfony 3 : 3.4

Architecture Symfony
* app : _config, resources:stocke les templates, erreurs personnalisées, etc._
* bin : _executables, console de symfony_
* src :
* * AppBundle
* * * Controller __C__
* * * * DefaultController
* * * Entity __M__
* * Resources : contenant les fichiers de description du modèle __M__: ORM Object Relationship Manager (Doctrine), description du modèle, routage, templates twigs_ __V__
* * Repository : va contenir les classes de requêtes_ __M__
* tests : _tests unitaires/fonctionnels_
* var : _cache & logs_
* vendor : _noyau, bibliothèques tierces (packagist), bibliothèques de l'appli._
* web : _point d'entrée public. S'appelle public dans les versions suivantes. CSS, JS..._



php bin/console pour lancer la console de symfony.

Avant Symfony 3.3, Symfony était composé de bundles : unités peu fonctionnelles de manière autonome. (MVC)

Dans le dossier src, on a un "AppBundle", appellé à disparaître dans les versions ultérieures.

Concept de "services" gérant tous les cas d'utilisations, traitement algorithmiques des données.

Schéma simple d'une requête
REQUÊTE => ROUTEUR => CONTRÔLEUR => MODELE -> Stockage
                    -> React, Angular, Tw  -> Requêtes
                                           -> [Persistance](quiver:///notes/8E89CBDD-7EF0-4447-A089-B5A493E7FEF9) (couche de). Permet à l'ORM de maintenir une cohérence entre l'application et les données stockées sur le disque
                                           

![IMG_20190812_105736.jpg](resources/54650D61DA69C85C2EF3009FA2047D95.jpg =3456x4608)

Comment Symfony affiche la première page d'accueil ? Grâce au DefaultController situé dans l'AppBundle/Controller (namespace AppBundle\Controller)

```php
<?php

namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class DefaultController extends Controller
{
    /**
     * @Route("/", name="homepage")
     */
    public function indexAction(Request $request)
    {
        // replace this example code with whatever you need
        return $this->render('default/index.html.twig', [
            'base_dir' => realpath($this->getParameter('kernel.project_dir')).DIRECTORY_SEPARATOR,
        ]);
    }
}
```

Ici, le routeur a déjà analysé la requête HTTP en amont. La variable $request est un objet contenant toute notre requête.

La méthode 'render' va construire la vue. 
1er paramètre : lien vers la vue. Celui-ci se trouve dans le dossier app/resources/views
2eme paramètre : tableau associatif. 'base_dir' va devenir une variable native dans notre template.

L'annotation @Route lie l'URL écrite dans le navigateur et le controlleur appelé par la suite.

#### Création d'une page "à propos"

Pour faire un lien vers une nouvelle page, on utilise la méthode 'render'. 

Par exemple, dans notre DefaultController :

```php
    /**
     * @Route("/about", name="about")
     */
public function aboutAction() {
        return $this->render('default/about.html.twig', [

        ]);
     }
```

On définit la route de notre page avec l'annotation @Route au-dessus de la méthode concernée.

Le "name" est important. Si on veut faire référence à cette route plus tard, on l'appellera grâce à ce "name" là.

Dans cette version, pas d'index.php, mais app.php (situé dans le dossier web)

On a aussi un point d'entrée 'app_dev.php', donc <dossier>/web/app_dev.php/about pour accéder à notre exemple.

Maintenant, si je souhaite accéder à cette URL : /about/{prenom}
On peut tout simplement envoyer des paramètres à notre méthode aboutAction()

TRES UTILE POUR DEBUGGER LES ROUTES :
 bin/console debug:router

```php
    /**
     * @param string $prenom
     * @param Request $request
     * @return \Symfony\Component\HttpFoundation\Response
     * @Route("/about/{prenom}", name="aboutname")
     */
   public function aboutController(string $prenom, Request $request) {
        var_dump($prenom);
        $prenom = ucfirst($prenom);
        return $this->render('default/about.html.twig', [
            'p' => $prenom
        ]);
        //Par ce mécanisme, je transmet la valeur de ma variable au template about
    }
```



(pour vider le cache, supprimer le cache dans le dossier var !)

[Twig](quiver:///notes/CDFDCD5D-97CC-4FAE-9B98-3B439ECCBDFF)

<link rel="stylesheet" href={{ asset('css/style.css') }} />aa









