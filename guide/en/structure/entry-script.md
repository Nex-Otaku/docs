# Entry Scripts

Entry scripts are the first step in the application bootstrapping process. An application (either
Web application or console application) has a single entry script. End users make requests to
entry scripts which instantiate application instances and forward the requests to them.

Entry scripts for Web applications must be stored under Web accessible directories so that they
can be accessed by end users. They are often named as `index.php`, but can also use any other names,
provided Web servers can locate them.

Entry script for console application is `./vendor/bin/yii`.

Entry scripts mainly perform the following work:


* Register [Composer autoloader](https://getcomposer.org/doc/01-basic-usage.md#autoloading);
* Obtain configuration;
* Use configuration to initialize dependency injection container;
* Call `Application::run()` to process the incoming request.


## Web Applications <span id="web-applications"></span>

The following is the code in the entry script for the application template:

```php
<?php
use hiqdev\composer\config\Builder;
use Yiisoft\Di\Container;
use Yiisoft\Yii\Web\Application;
use Yiisoft\Yii\Web\ServerRequestFactory;

require_once dirname(__DIR__) . '/vendor/autoload.php';

// Don't do it in production, assembling takes it's time
Builder::rebuild();

$container = new Container(require Builder::path('web'));

$request = $container->get(ServerRequestFactory::class)->createFromGlobals();
$container->get(Application::class)->handle($request);
```


## Console Applications <span id="console-applications"></span>

Similarly, the following is the code for the entry script of a console application:

```php
#!/usr/bin/env php
<?php

use hiqdev\composer\config\Builder;
use Yiisoft\Di\Container;
use Yiisoft\Yii\Console\Application;

(static function () {
    $cwd = getcwd();

    $possibleAutoloadPaths = [
        // running from project root
        $cwd . '/vendor/autoload.php',
        // running from project bin
        dirname($cwd) . '/autoload.php',
        // local dev repository
        dirname(__DIR__) . '/vendor/autoload.php',
        // dependency
        dirname(__DIR__, 4) . '/vendor/autoload.php',
    ];
    $autoloadPath = null;
    foreach ($possibleAutoloadPaths as $possibleAutoloadPath) {
        if (file_exists($possibleAutoloadPath)) {
            $autoloadPath = $possibleAutoloadPath;
            break;
        }
    }

    if ($autoloadPath === null) {
        $message = "Unable to find vendor/autoload.php in the following paths:\n\n";
        $message .= '- ' . implode("\n- ", $possibleAutoloadPaths) . "\n\n";
        $message .= "Possible fixes:\n";
        $message .= "- Install yiisoft/console via Composer.\n";
        $message .= "- Run ./yii either from project root or from vendor/bin.\n";
        fwrite(STDERR, $message);
        exit(1);
    }
    require_once $autoloadPath;

    $container = new Container(require Builder::path('console'));

    $container->get(Application::class)->run();
})();
```
