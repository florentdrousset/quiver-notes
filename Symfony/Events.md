[Doc Symfony Event Dispatcher](https://symfony.com/doc/current/components/event_dispatcher.html)
[Doc Symfony Event Listeners & Subscribers](https://symfony.com/doc/current/event_dispatcher.html)

Exemple du `httpKernelComponent`:
Quand un objet `Response` est créé, il pourrait être utile d'autoriser d'autre éléments du système de le modifier. Pour rendre cela possible, Symfony dispose d'un évènement `kernel.response`. Fonctionnement :

1. Un listener dit au __dispatcher__ qu'il veut se mettre à écouter l'évènement `kernel.response`.
2. Le kernel de Symfony dit au __dispatcher__ de dispatcher l'évènement `kernel.response`, en lui passant un objet `Event` qui aura accès à l'objet `Response`.
3. Le dispatcher envoie une notif à tous ceux qui écoutent l'évènement `kernel.reponse`, autorisant chacun d'entre eux à modifier l'objet `Response`.


Dans notre dossier Event, on créera une classe pour le type d'évènement que l'on veut créer :
ex : OrderPlacedEvent, RegisterEvent, LoginEvent...

```php
<?php


namespace App\Events;

use App\Entity\Event;
use App\Entity\User;

class BookingEvent extends Event
{
    public const NAME = 'book.booking';

    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function getUser()
    {
        return $this->user;
    }
}
```

Ici, on a créé un getter pour que notre subscriber puisse accéder à notre User.

à l'endroit où on veut appeler notre event :

```php
        $event = new BookingEvent($user);
        $dispatcher = new EventDispatcher();
        $dispatcher->dispatch($event, BookingEvent::NAME);
```