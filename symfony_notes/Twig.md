## TWIG

Twig est un moteur de rendu utilisé par Symfony. 
Pour reprendre notre exemple précédent, on peut afficher Bonjour {{ p }} dans notre vue associée !
On peut tout à fait mélanger twig et HTML.

```twig
{{ p }}
{{ objet.prenom }}
```

Directives Twig :

```twig
{% if --- %}
{% endif %}
```

Commentaires :

{# commentaires #}

Possibilité d'avoir des modificateurs avec le pipe :

```twig
 <p>écrit par {{ item.auteur }} en {{ item.parution|date('Y/m/d') }}</p>
```

On peut faire hériter un template d'un autre avec {% extends 'fichier' %}
De base, 'index.html.twig' hérite de base.html.twig. 

Exercice : route avec un ID pour faire la fiche d'un livre. On veut afficher dans la page l'id en question. On veut avoir un header et un footer, chacun dans des fichiers séparés. On voudrait aussi avoir une barre latérale dans laquelle on affichera plusieurs livres qui ont le même auteur. 
Directive twig include.
{% include %}
On peut faire appel à un contrôleur directement à partir de twig, grâce à la directive {% controller %}

On souhaite que Twig redéclenche un cycle d'exécutions :

```twig
{{ render(controller('App\\Controller\\DefaultController::relatedAction',
        'name': { books.auteur })
    ) }}
```

### Mécanisme des requêtes via Twig

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

Twig récupère en fait les informations minimum de 'Artist', et récupèrera les infos à la volée via Doctrine au moment ou notre boucle Twig sera lue.