## Lundi 7 Octobre

https://juliendubreuil.fr/blog/drupal/hooks-drupal7/

## Vendredi 4 Octobre

[Ajouter un asset externe]()

```yaml
nihongaku-fonts:
  version: VERSION
  css:
    theme:
      'https://fonts.googleapis.com/css?family=Kosugi+Maru|M+PLUS+Rounded+1c|Montserrat|Open+Sans|Roboto&display=swap': { type: external, minified: true }
```

## Jeudi 3 Octobre

#### Création de module

Pour créer un module, on doit d'abord créer son dossier qui définira son "machine name".
Source : https://www.drupal.org/docs/8/creating-custom-modules/naming-and-placing-your-drupal-8-module

Pour un module appelé hello_world, deux paths au choix :
`/modules/custom/hello_world`
ou
`/sites/all/modules/hello_world`

On doit ensuite créer un fichier info.yaml pour faire savoir à Drupal que notre module existe. Exemple :

```yaml
name: Hello World Module
description: Creates a page showing "Hello World".
package: Custom

type: module
core: 8.x
```

Les trois premières lignes sont dédiées à l'UI d'administration. Le 'package' permet de grouper des modules ensemble quand on veut activer ou désactiver des modules.

Pour installer la console Drupal :
https://drupalconsole.com/docs/en/getting/launcher

Pour installer un module :
`drupal module:install admin_toolbar`

On peut générer un bloc à partir d'un module grâce à la commande :
`drupal generate:plugin:block`

## Lundi 30 Septembre

Comment [définir notre propre thème](https://www.drupal.org/docs/8/theming-drupal-8/defining-a-theme-with-an-infoyml-file)

Il nous faut toujours un `fichier.info.yml`.

Dans ce cas là, on va créer le thème 'academy'.
Indispensable dans le YAML :

```yaml
name : academy
type: theme
core: 8.x
# optionnel : on peut hériter d'un autre thème :
base theme : classy
# si on ne met rien ici, il héritera par défaut du thème 'stable'
regions :
  header : 'header'
  footer : 'footer'
# si on définit ne serait-ce qu'une seule région, on écrase toutes les régions du thème dont on a hérité.
```

On définira nos assets dans le fichier `academy.libraries.yml`

```yaml
global-styling : 
  version : VERSION
  css :
    theme :
      css/style.css : {}
  
```

[Bonne explication du dossier templates](https://www.itss.paris/blog/developper-un-theme-sur-drupal-8) qui contient le code HTML de notre thème.

Pour éditer les régions d'un thème Drupal, il faut :
* ajouter les méta-données du thème dans notre fichier `xxx.info.yaml`
* Editer le fichier `page.html.twig` en conséquence

Note: Pour [Changer la page d'accueil](quiver:///notes/B829D7E2-F0A2-4CD6-A0E0-89ACA591FDDD), aller dans Configuration > Système > Paramètres de base du site

Pour [Désactiver le cache (mode développement)](quiver:///notes/0A65653D-0064-4F61-808A-CEB90A9F6E06) : `drupal site:mode dev`



## Jeudi 26 Septembre

Ajouter des champs au formulaire d'inscription :
admin/config/people/accounts/fields/

## Mercredi 25 Septembre

(Adding stylesheets / JS)[https://www.drupal.org/docs/8/theming/adding-stylesheets-css-and-javascript-js-to-a-drupal-8-theme#define]

Module de slide : Slick -> Blazy
Owl

Utilisation des webforms :
Installation du module webform. Pour y accéder, l'url est `admin/structure/webform`

## Mardi 24 Septembre

On continue sur le TP.
Petit tuto pour (AddToAny)[https://alvinalexander.com/misc/drupal-how-to-configure-addtoany-drupal-8-block] : penser à placer les blocs.

# Drupal 

## Lundi 23 Septembre 2019

Pour créer des modules, on créera au préalable un dossier sites > all > modules.
On crée un dossier 'academy' puis un module 'academy.info.yml' à l'intérieur de celui-ci.

Tous les modules doivent finir par .info.yaml.

Effacer tous les caches :
Configuration > Développent > Performance > Effacer les caches

```yaml
# Exemple de description minimale

name: academy
description: formation #optionnel
type: module
core: 8.x
```

On crée ensuite un fichier `academy.routing.yml`



Module utile en tant qu'admin :
[admin toolbar](https://www.drupal.org/project/admin_toolbar)

TP N°1 : Le Bon Logement

* Installation d'un thème https://www.drupal.org/project/nexus
* 







Drupal is a flexible CMS based on the LAMP stack, with a modular design allowing features to be added and removed by installing and uninstalling modules, and allowing the entire look and feel of the website to be changed by installing and uninstalling themes. The base Drupal download, known as Drupal Core, contains the PHP scripts needed to run the basic CMS functionality, several optional modules and themes, and many JavaScript, CSS, and image assets. Many additional modules and themes can be downloaded from the Drupal.org website.

### Un peu de vocabulaire

Dans Drupal, un contenu est appelé _node_. Un _node_ est identifiable via son ID unique. Il est caractérisé par un _content type_ et des métadonnées telles qu'un auteur, une date de création, etc...
Un _node_ aura une URL au sein de notre site. 
Un _content_ est un assemblage de _fields_. (booléen, nombre, file field, image field, list, etc...)

Par défaut, Drupal fournit un certains nombre de types de champs. Vous pouvez ensuite ajouter des modules complémentaires définissant des types de champs pour à peu près tout ! Pour une URL, email, vidéo, QCM, un sondage Doodle, etc...



### Installation
[suivi du tuto Installation d'une plateforme de test de drupal-addict](https://drupal-addict.com/installation-d-une-plateforme-de-test)

On peut tout simplement télécharger Drupal sur le site officiel et le dézipper dans notre localhost.
La méthode recommandée est tout de même [d'utiliser composer](https://www.drupal.org/docs/8/install/step-1-get-the-code)

Pour [installer Drupal avec composer](https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies) :

```js
composer create-project drupal-composer/drupal-project:8.x-dev my_site_name_dir --no-interaction
```

