## Mer. 7 Août 2019

Autoloader fait maison :

```php
class Autoloader
{
    static function register() {
        spl_autoload_register(array(__CLASS__, 'autoload'));
    }

    static function autoload($class) {
        $class = str_replace('\\', '/', $class);
        require ROOT_PATH . '/src/' . $class .'.php';
    }
}
```

index.php :

```php
const ROOT_PATH = __DIR__;
require 'app/Autoloader.php';
Autoloader::register();
use Exemple\A;

$test = new A;
```

penser à ajouter le mot clef "final" pour les classes qui ne seront pas des classes parent

