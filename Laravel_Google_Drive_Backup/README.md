
##### 1. Create Project on Google Colud console
##### 2. Goto Library and Enaled Google Drive API
##### 3. Goto Oauth Consent screen and set (external) Project configuration
##### 5. Goto credentials Create Oauth Client id
* --> use this redirect url https://developers.google.com/oauthplayground


##### 6. Copy Client and secret id and goto this url and set client and secret 
* https://developers.google.com/oauthplayground
* click settings icon and check (Use your own OAuth credentials) and set client id and secret id in textboxs

##### 6. Goto Oauth Consent Screen Audience/Publishing status into publich

##### 7. Find Drive API v3 and check related url then click authorize
* If you see this, it means your test app has successfully been created and Google login to the API is working.
* You need to grant all permissions for Drive, after which you will be redirected back to the OAuth 2.0 Playground, where you need to click on 'Exchange authorization code for tokens' to generate the refresh token and access token

##### 8. Goto Google Drive
Create LaravelBackupProject Folder in Google Drive
and copy folder id 

##### 9. Create Project and install packages
```bash
composer require spatie/laravel-backup
```

```bash
php artisan vendor:publish --provider="Spatie\Backup\BackupServiceProvider"
```

#### 10. Goto laravel project Config/backup.php Add google perameter
```bash
'disks' => [
    'google',
    'local',
],
```

```bash
composer require masbug/flysystem-google-drive-ext
```

```bash
php artisan make:provider loadGoogleStorageDriver
```

##### 11. Goto config/filesystems.php
```bash
'disks' => [
    'google' => [
        'driver' => 'google',
        'clientId' => env('GOOGLE_DRIVE_CLIENT_ID'),
        'clientSecret' => env('GOOGLE_DRIVE_CLIENT_SECRET'),
        'refreshToken' => env('GOOGLE_DRIVE_REFRESH_TOKEN'),
        'folderId' => env('GOOGLE_DRIVE_FOLDER_ID'), // Optional
    ],
],
```


##### 12. Provider
```bash
<?php

namespace App\Providers;

use Google_Client;
use Google_Service_Drive;
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Storage;
use Masbug\Flysystem\GoogleDriveAdapter;
use League\Flysystem\Filesystem;


class loadGoogleStorageDriver extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        //
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        Storage::extend('google', function ($app, $config) {
            $client = new Google_Client();
            $client->setClientId($config['clientId']);
            $client->setClientSecret($config['clientSecret']);
            $client->refreshToken($config['refreshToken']);

            $service = new Google_Service_Drive($client);

            $adapter = new GoogleDriveAdapter($service, $config['folderId'] ?? null);

            return new Filesystem($adapter);
        });
    }
}

```






##### Set .ENV
```bash
GOOGLE_DRIVE_CLIENT_ID=your_client_id
GOOGLE_DRIVE_CLIENT_SECRET=your_client_secret
GOOGLE_DRIVE_REFRESH_TOKEN=your_refresh_token
GOOGLE_DRIVE_FOLDER_ID=your_folder_id  # optional
```