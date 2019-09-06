```php

 <?php
     public function findByEvents() {
         //Requete DQL classique
         /*$q = $this->getEntityManager();
         $query = $q->createQuery('SELECT a.name FROM App\Entity\Event e JOIN e.artist_id a GROUP BY e.artist_id ORDER BY COUNT(e.artist_id) DESC')
         ->setMaxResults(10);
         $result = $query->getResult();*/

         /*return $result;*/

         //Version QueryBuilder
         return $this->createQueryBuilder('e')
             ->select('COUNT(e.artist_id) as maxE, a.name')
             ->innerJoin('App\Entity\Artist', 'a')
             ->where('a.id = e.artist_id')
             ->groupBy('e.artist_id')
             ->orderBy('maxE', 'DESC')
             ->setMaxResults(2)
             ->getQuery()
             ->getResult();
     }

 ```


```

[Tuto Query Builder en français](https://zestedesavoir.com/tutoriels/1713/doctrine-2-a-lassaut-de-lorm-phare-de-php/exploiter-une-base-de-donnees-avec-doctrine-2/a-la-rencontre-du-querybuilder-1/)

```php
  32 $qexpr = $q1->expr()
  33 $subrequest = $q1->select('a.id')
  34 ->where($qexpr->like('a.pays, $qexpr->'literal('USA')));
  35 ->where($qexpr->in('a.id', $subrequest->getDQL()))
```

Dans notre entité 'Artist', on a bien un champ 'events' qui assure la relation OneToMany avec la table 'Events'. Mais celle-ci n'apparait pas dans _PHPMyAdmin_.

Requête pour chercher tous les artistes ayant fait le plus d'évènements et provenant de tel pays :

```php
     public function findByExampleField()
     {
         // SOUS-REQUÊTE
         $q1 = $this->createQueryBuilder('x');
         $qexpr = $q1->expr();
         $subrequest = $q1->select('x.id')
             ->where($qexpr->like('x.country', $qexpr->literal('USA')));

         // REQUÊTE PRINCIPALE
         return $this->createQueryBuilder('a')
             ->select('a.name')
             ->innerJoin('a.events', 'e')
             ->where($qexpr->in('a.id', $subrequest->getDQL())) // INJECTION SOUS-REQUÊTE
             ->groupBy('a.id')
             ->getQuery()
             ->getResult()
             ;
     }
```

