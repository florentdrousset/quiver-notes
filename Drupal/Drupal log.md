# Drupal 

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