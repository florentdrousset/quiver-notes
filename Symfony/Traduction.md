   1 # Traductions
-  2
|  3 1. paramètres
|  4 2. routes
|  5 3. -> templates twig
|  6 -> controllers/services
|  7 4. Importer la traduction
|  8 5. traduction du contenu avec les extensions Doctrine(__translatable__)
|  9
| 10 Commençons par le 3 (Twig)
| 11
| 12 Si on a un template :
| 13 ```twig
| 14
| 15 <p> {% trans %}Bienvenue sur xxx.com {% endtrans %}</p>
| 16
| 17 ```
| 18
| 19 Donc, penser à utiliser la balise `trans` partout où on aura une traduction à faire.
| 20
| 21 On peut accéder au debug avec la commande `bin/console debug:translation fr` par exemple. (ou `en`, `zh`, etc...)
| 22
| 23 `bin/console translation:update fr` pour ajouter la traduction française.
| 24
| 25 Les deux grands formats de traduction sont :
| 26 * xlf
| 27 * po
| 28
| 29 A la racine, on a un dossier translations/ qui contient tous les fichiers de traduction.
| 30
| 31 On voudrait avoir une politique d'URL qui intègre la langue. Les deux méthodes les plus répandues sont :
| 32 * utilisation de sous-domaines (fr.wikipedia.org, en.wikipedia.org...)
| 33 * utilisation d'URL multi-lingues (méthode que l'on va utiliser)
| 34
| 35
| 36 _exemple_
| 37 `index.php`   /artist/25
| 38 on va intercaler la langue voulue dans l'URL :
| 39 `index.php/fr/artiste/25`
| 40
| 41 Pour cela, on va utiliser la variable `{_locale}`
| 42
| 43 Si la langue n'existe pas, on se repliera sur une langue par défaut paramétrée dans le site.
| 44
| 45 On va remplacer nos `@Route ('/artist')` par `@Route'/{_locale}/artist`
| 46
| 47 On a un fichier `translation.yaml` pour choisir la langue par défaut.