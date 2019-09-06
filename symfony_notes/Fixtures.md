   1 Exemple d'une classe de fixtures :
   2 ```php
   3 <?php
   4 namespace App\DataFixtures;
   5
   6 use App\Entity\Artist;
   7 use App\Entity\Event;
   8 use Doctrine\Bundle\FixturesBundle\Fixture;
   9 use Doctrine\Common\Persistence\ObjectManager;
  10 use Faker;
  11
  12 class AppFixtures extends Fixture
  13 {
  14     public function load(ObjectManager $manager)
  15     {
  16         // $product = new Product();
  17         // $manager->persist($product);
  18         $faker = Faker\Factory::create();
  19         for($i = 0 ; $i < 10; $i++)
  20         {
  21             $artist = new Artist();
  22             $artist->setName($faker->name);
  23             $artist->setCountry($faker->country);
  24             $artist->setGenre($faker->randomElement(array('Rock', 'Jazz', 'Soul', 'Ambient')));
  25             $artist->setDescription($faker->catchPhrase);
  26             $manager->persist($artist);
  27         }
  28         $manager->flush();
  29     }
  30 }
  31
  32 ```
  33
  34 Pour charger les fixtures :
  35 ```bash
  36 php bin/console doctrine:fixtures:load
  37 ```
  38
  39 On peut aussi ajouter le flag --append pour éviter de purger toute la DB.