

### Caution
Use the DebugBar only in development. Do not use Debugbar on publicly accessible websites, as it will leak information from stored requests (by design).

### Warning
It can also slow the application down (because it has to gather and render data). So when experiencing slowness, try disabling some of the collectors.

#### This package includes some custom collectors:
* QueryCollector: Show all queries, including binding + timing
* RouteCollector: Show information about the current Route.
* ViewCollector: Show the currently loaded views. (Optionally: display the shared data)
* EventsCollector: Show all events
* LaravelCollector: Show the Laravel version and Environment. (disabled by default)
* SymfonyRequestCollector: replaces the RequestCollector with more information about the request/response
* LogsCollector: Show the latest log entries from the storage logs. (disabled by default)
* FilesCollector: Show the files that are included/required by PHP. (disabled by default)
* ConfigCollector: Display the values from the config files. (disabled by default)
* CacheCollector: Display all cache events. (disabled by default)

https://github.com/barryvdh/laravel-debugbar

```bash
composer require barryvdh/laravel-debugbar --dev
```
* The Debugbar will be enabled when APP_DEBUG is true in .env file.


#### Goto bootstrap/providers.php/ and add
```bash
<?php
return [
    // ... existing providers ...
    Barryvdh\Debugbar\ServiceProvider::class,
];
```
#### app/Providers/AppServiceProvider.php ke register() method mein ye code add kry
```bash
public function register(): void
    {
        $loader = \Illuminate\Foundation\AliasLoader::getInstance();
        $loader->alias('Debugbar', \Barryvdh\Debugbar\Facades\Debugbar::class);
    }
``` 

##### if you run Command to create file config/debugbar.php
```bash
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

Abb ap Project Run kr ke Check kr sakty hain