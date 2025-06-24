

#### Install
```bash
composer require laravel/breeze --dev
php artisan breeze:install blade

php artisan migrate
npm install && npm run dev
```

### Breeze Install Karne Ke Baad Kya Hota Hai?
### 1. New Folders & Files Create Honge:

#### 📁 routes/web.php
Routes me login, register, dashboard waghera ke routes add ho jate hain.

#### 📁 app/Http/Controllers/Auth/
Authentication se related controllers (LoginController, RegisteredUserController, etc.)

#### 📁 resources/views/auth/
Login, Register, Forgot Password waghera ke Blade views.

#### 📁 resources/views/layouts/
Common layout file like app.blade.php.

#### 📁 resources/views/dashboard.blade.php
Login hone ke baad dikhne wala dashboard page.

#### 📁 app/Models/User.php
Default User model jisme Laravel Breeze ka authentication ka logic hota hai.

#### 📁 routes/auth.php
Breeze ke authentication routes is file me define hote hain aur web.php se include hote hain.

#### 📁 database/migrations/
Users table, password resets etc. ke liye required migrations pehle se hote hain, lekin Breeze me kuch bar update bhi ho sakte hain.





php artisan make:migration create_admins_tabel

```bash
 Schema::create('admins_tabel', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->string('token')->nullable();
            $table->timestamps();
        });
```

```bash
php artisan make:model Admin
```

```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class Admin extends Authenticatable
{
    use HasFactory, Notifiable;
  
    protected $quard = 'admin';
    
    protected $fillable = [
        'name',
        'email',
        'password',
        'token'
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
```

#### config/auth.php
* Add This Code
```bash
'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',
        ],

    ],
```   

```bash
   'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => env('AUTH_MODEL', App\Models\User::class),
        ],

        'admins' => [
            'driver' => 'eloquent',
            'model' => env('AUTH_MODEL', App\Models\Admin::class),
        ],
    ],
```

#### Create MiddleWare
```bash
php artisan make:middleware Admin
```

* Register Middleware in bootstrap/app.php
```bash
->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'admin' => Admin::class,
        ]);
    })
```

### Admin MiddleWare 
```bash
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
use Illuminate\Support\Facades\Auth;

class Admin
{

    public function handle(Request $request, Closure $next): Response
    {
        if (!Auth::guard('admin')->check()) {

            return redirect()->route('admin_login')->with('error', 'you do not have permission to access this page');
        }

        return $next($request);
    }
}

```

#### Create Seeder
```bash
php artisan make:seeder AdminSeeder
```
```bash
<?php

namespace Database\Seeders;

use App\Models\Admin;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class AdminSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $obj = new Admin();
        $obj->name = 'Admin';
        $obj->email = 'admin@gmail.com';
        $obj->password = Hash::make('1234');
        $obj->save();
    }
}
```

* AdminSeeder ko DatabaseSeeder.php me include kiya hai:
```bash
  public function run(): void
    {
         $this->call(AdminSeeder::class);
    }
 ```





