
```bash
composer require spatie/laravel-backup
```

```bash
php artisan vendor:publish --provider="Spatie\Backup\BackupServiceProvider"
```


#### Set env:
```bash

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravelbackup
DB_USERNAME=root
DB_PASSWORD=


MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=xxxxxxxxxxxxxxxx
MAIL_PASSWORD=xxxxxxxxxxxxxxxxxx
MAIL_FROM_ADDRESS=xxxxxxxxxxxxxxxxxxxxx
MAIL_FROM_NAME="${APP_NAME}"
```
#### condig/database.php
```bash
 'mysql' => [
            'driver' => 'mysql',
            'url' => env('DB_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => env('DB_CHARSET', 'utf8mb4'),
            'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
            'dump' => [
                'dump_binary_path' => 'C:\xampp\mysql\bin',
                'use_single_transaction',
                'timeout' => 60 * 5,
            ],
        ],
```       

```bash
php artisan backup:run
```

#### Make Backup this location:
```bash
storage\app\private\Laravel
```