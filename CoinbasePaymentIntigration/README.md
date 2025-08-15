

### 1. Create Api


### 2---Install
```bash
composer require firebase/php-jwt phpseclib/phpseclib
```


### 3. Generate a JWT for Authentication
* Create a new file in your Laravel project, e.g., app/Services/CoinbaseJwtService.php:
```bash
<?php

namespace App\Services;

use Firebase\JWT\JWT;
use phpseclib3\Crypt\EC;
use Illuminate\Support\Facades\Log;

class CoinbaseJwtService
{
    public function generateJwt($requestMethod, $requestPath)
    {
        $keyName = 'organizations/b678bb46-a854-4525-af76-a35d1e44bbf0/apiKeys/5c033155-d84b-46e2-bd33-44a44ac6cb3d';
        $keySecret = <<<EOD
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIJXky9IOWnhp3utY6axxCSQX/UQCFvEOr32E0jMRRvbUoAoGCCqGSM49
AwEHoUQDQgAEyTuUtzsXevGFBTuRPAOtP2nOdp3j0GGTFcqg+Rgha28Ax0hhahk/
JCQ7OCXRYMIjkpOUMD5otExYRmZz6JigUQ==
-----END EC PRIVATE KEY-----
EOD;
        # test mode
        $requestHost = 'api-public.sandbox.coinbase.com';
        # $requestHost = 'api.coinbase.com'; //production level

        // URI construct karein
        $uri = "$requestMethod $requestHost$requestPath";

        // JWT payload tayyar karein
        $currentTime = time();
        $payload = [
            'sub' => $keyName,
            'iss' => 'coinbase-cloud',
            'nbf' => $currentTime,
            'exp' => $currentTime + 120,
            'uri' => $uri,
        ];

        // Headers tayyar karein
        $headers = [
            'kid' => $keyName,
            'nonce' => bin2hex(random_bytes(16)),
        ];

        // ECDSA private key load karein
        try {
            if (!str_contains($keySecret, '-----BEGIN EC PRIVATE KEY-----')) {
                Log::error("Coinbase JWT: Invalid key format, ECDSA PEM expected");
                throw new \Exception("Invalid key format: ECDSA PEM key required");
            }
            $privateKey = EC::loadPrivateKey($keySecret);
            $jwt = JWT::encode($payload, $privateKey, 'ES256', null, $headers);
            return $jwt;
        } catch (\Exception $e) {
            Log::error("Coinbase JWT Generation Failed: " . $e->getMessage() . " | Key Preview: " . substr($keySecret, 0, 30) . "...");
            throw new \Exception("JWT generate nahi hua: " . $e->getMessage());
        }
    }
}

```

### Register the Service
* Add the service to Laravel’s service container by creating a service provider or binding it in app/Providers/AppServiceProvider.php:
```bash
<?php

namespace App\Providers;

use App\Services\CoinbaseJwtService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(CoinbaseJwtService::class, function () {
            return new CoinbaseJwtService();
        });
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        //
    }
}
```

### Create a Payment Controller
```bash
php artisan make:controller PaymentController
```
```bash
<?php

namespace App\Http\Controllers;

use App\Services\CoinbaseJwtService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class PaymentController extends Controller
{
    protected $jwtService;

    public function __construct(CoinbaseJwtService $jwtService)
    {
        $this->jwtService = $jwtService;
    }

    public function getAccounts()
    {
        try {
            // Generate JWT
            $jwt = $this->jwtService->generateJwt('GET', '/v2/accounts');

            // Make API request
            $response = Http::withHeaders([
                'Authorization' => "Bearer $jwt",
                'Content-Type' => 'application/json',
                'Accept' => 'application/json',
            ])->get("https://api.coinbase.com/v2/accounts");

            if ($response->successful()) {
                return response()->json($response->json());
            } else {
                Log::error("Coinbase API Error: " . $response->body());
                return response()->json(['error' => 'Failed to fetch accounts'], 500);
            }
        } catch (\Exception $e) {
            Log::error("Coinbase API Request Failed: " . $e->getMessage());
            return response()->json(['error' => 'Server error'], 500);
        }
    }
}
```

### Routes
```bash
Route::get('/coinbase/accounts', [PaymentController::class, 'getAccounts']);
```
* php artisan serve
* http://localhost:8000/coinbase/accounts





