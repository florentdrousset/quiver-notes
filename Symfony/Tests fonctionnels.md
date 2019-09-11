   1 # Tests Fonctionnels
-  2 WebTestCase permet de simuler un navigateur.  Avec Panther, on peut faire des classes de tests qui héritent de PantherTestCase.  L'avantage de PantherTestCase est qu'on va communiquer avec un vrai navigateur (chrome). (Bientôt Firefox) Permet d'exécuter du JS, etc,  -    contrairement à WebTestCase. Sinon les API sont exactement les mêmes.  Sur le repo GitHub, on a un driver nommé _WebDriver_ qui assure la communication entre Symfony et Chrome.
|  3
|  4 On a 3 choses dans Symfony pour les tests fonctionnels :
|  5 1. PhpUNIT
|  6 2. Client
|  7 3. Crawler
|  8
|  9 ### Client
+ 10 +----  4 lines: Le __client__ s'occupe de la partie _requête HTTP_. -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
2 14 ### Crawler
- 15 Le __crawler__ navigue dans la page et trouve des élements. Genre de _DOM_ qui va nous permettre de naviguer, trouver des éléments en naviguant dans des noeuds.
3 16
3 17 L'écrite des tests va être la combinaison de tout ça.
3 18
3 19 ```php
3 20 <?php
3 21
3 22 class FunctionnalTest() o
3 23     $client = $static::createClient();
3 24     $crawler = $client->request()
3 25 }
3 26
3 27
3 28 ```
3 29
3 30 Dans Symfony, on a la classe DOMCrawler qui permet d'accéder à toutes les fonctionnalités de navigation dans la page.
3 31 On peut donc afficher une page, et chercher des éléments dans cette page.
3 32
3 33 __Exercice__ :
3 34
3 35 On arrive sur la page d'accueil. Liste de liens (évènements par exemple).
3 36 Sur chaque lien, on est capable de récupérer le lien qui correspond au premier évènement (par exemple). On est capable d'aboutir sur cette page, et de vérifier qu'on a bien un titre qui correspond au nom de l'artiste.
3 37
3 38 On peut créer un test fonctionnel avec la commande `