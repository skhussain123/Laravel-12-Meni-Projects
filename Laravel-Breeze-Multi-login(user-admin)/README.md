

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

 #### Create Admin Controller
 ```bash
 php artisan make:Controller Admin/AdminController
 ```

#### Create two View file 
* view/admin/dashboard
* view/admin/login
* view/admin/reset-password.blade.php
* view/admin/forget-password.blade.php


#### Admin Routes
```bash
// Admin Routes
Route::middleware('admin')->prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard'])->name('admin_dashboard');
});


Route::prefix('admin')->group(function () {
    Route::get('/login', [AdminController::class, 'login'])->name('admin_login');
    Route::post('/login-submit', [AdminController::class, 'login_submit'])->name('admin_submit');
    Route::get('/logout', [AdminController::class, 'logout'])->name('admin_logout');

    Route::get('/forget-password', [AdminController::class, 'ForgetPassword'])->name('forgetpassword');
    Route::post('/forget-password-submit', [AdminController::class, 'admin_forget_password_post'])->name('admin_forget_password_submit');
    Route::get('/reset-password/{token}/{email}', [AdminController::class, 'reset_password'])->name('admin_reset_password');
    Route::post('/forget-reset-password-submit', [AdminController::class, 'admin_reset_password_post'])->name('admin_reset_password_submit');
});
```

#### view/admin/dashboard.blade.php
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Admin DashBoard</title>
</head>
<body>
    <h4>Welcome Admin Dashboard</h4>
    <a href="{{route('admin_logout')}}">Logout</a>
</body>
</html>
```

#### view/admin/login.blade.php
```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Login</title>

    <!-- Bootstrap 5 CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Custom CSS -->
    <style>
        body {
            background: #f8f9fa;
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .login-box {
            background: #fff;
            border-radius: 10px;
            padding: 40px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 400px;
        }

        .login-box h4 {
            margin-bottom: 25px;
            font-weight: 600;
            text-align: center;
        }

        .form-control {
            margin-bottom: 15px;
        }

        .form-text {
            text-align: right;
        }

        .alert {
            padding: 8px 12px;
            margin-bottom: 10px;
        }
    </style>
</head>

<body>

    <div class="login-box">
        <h4>Admin Login</h4>

        {{-- Session Messages --}}
        @if (Session::has('success'))
            <div class="alert alert-success">{{ Session::get('success') }}</div>
        @endif

        @if (Session::has('error'))
            <div class="alert alert-danger">{{ Session::get('error') }}</div>
        @endif

        {{-- Validation Errors --}}
        @if ($errors->any())
            <div class="alert alert-danger">
                <ul class="mb-0">
                    @foreach ($errors->all() as $error)
                        <li style="font-size: 14px">{{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif

        <form action="{{ route('admin_submit') }}" method="POST">
            @csrf
            <input type="email" name="email" class="form-control" placeholder="Email">
            <input type="password" name="password" class="form-control" placeholder="Password">

            <div class="form-text mb-3">
                <a href="{{ route('forgetpassword') }}">Forgot Password?</a>
            </div>

            <button type="submit" class="btn btn-primary w-100">Login</button>
        </form>
    </div>

</body>

</html>
```

#### view/admin/reset-password.blade.php
```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Reset Password</title>

    <!-- Bootstrap 5 CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Custom CSS -->
    <style>
        body {
            background: #f1f3f5;
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .reset-box {
            background: #ffffff;
            padding: 35px;
            border-radius: 10px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.08);
            max-width: 450px;
            width: 100%;
        }

        .reset-box h4 {
            text-align: center;
            font-weight: 600;
            margin-bottom: 25px;
        }

        .form-control {
            margin-bottom: 15px;
        }

        .alert {
            padding: 10px;
            font-size: 14px;
        }

        .form-footer {
            text-align: center;
            margin-top: 10px;
        }
    </style>
</head>

<body>

    <div class="reset-box">
        <h4>Reset Password</h4>

        {{-- Session Messages --}}
        @if (Session::has('success'))
            <div class="alert alert-success">{{ Session::get('success') }}</div>
        @endif

        @if (Session::has('error'))
            <div class="alert alert-danger">{{ Session::get('error') }}</div>
        @endif

        {{-- Validation Errors --}}
        @if ($errors->any())
            <div class="alert alert-danger">
                <ul class="mb-0">
                    @foreach ($errors->all() as $error)
                        <li style="font-size: 14px">{{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif

        <form action="{{ route('admin_reset_password_submit') }}" method="POST">
            @csrf
            <input type="hidden" name="token" value="{{ $token }}">
            <input type="hidden" name="email" value="{{ $email }}">

            <input type="password" name="password" class="form-control" placeholder="New Password" required>
            <input type="password" name="password_confirmation" class="form-control" placeholder="Confirm New Password"
                required>

            <button type="submit" class="btn btn-primary w-100">Reset Password</button>
        </form>

        <div class="form-footer">
            <a href="{{ route('admin_login') }}">← Back to Login</a>
        </div>
    </div>

</body>

</html>
```

#### view/admin/forget-password.blade.php
```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Forget Password</title>

    <!-- Bootstrap 5 CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Custom CSS -->
    <style>
        body {
            background: #f1f3f5;
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .forget-box {
            background: #fff;
            padding: 35px;
            border-radius: 10px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
            max-width: 400px;
            width: 100%;
        }

        .forget-box h4 {
            margin-bottom: 20px;
            text-align: center;
            font-weight: bold;
        }

        .form-control {
            margin-bottom: 15px;
        }

        .form-footer {
            text-align: center;
            margin-top: 10px;
        }

        .alert {
            padding: 10px;
            font-size: 14px;
        }
    </style>
</head>

<body>

    <div class="forget-box">
        <h4>Forgot Password</h4>

        {{-- Session Messages --}}
        @if (Session::has('success'))
            <div class="alert alert-success">{{ Session::get('success') }}</div>
        @endif

        @if (Session::has('error'))
            <div class="alert alert-danger">{{ Session::get('error') }}</div>
        @endif

        {{-- Validation Errors --}}
        @if ($errors->any())
            <div class="alert alert-danger">
                <ul class="mb-0">
                    @foreach ($errors->all() as $error)
                        <li style="font-size: 14px">{{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif

        <form action="{{ route('admin_forget_password_submit') }}" method="POST">
            @csrf
            <input type="email" name="email" class="form-control" placeholder="Enter your email" required>

            <button type="submit" class="btn btn-primary w-100">Send Reset Link</button>
        </form>

        <div class="form-footer">
            <a href="{{ route('admin_login') }}">← Back to Login</a>
        </div>
    </div>

</body>

</html>
```

#### Admin/AdminController
```bash
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Mail\WebsiteMail;
use App\Models\Admin;
use Illuminate\Support\Facades\Auth;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Mail;

class AdminController extends Controller
{
    public function dashboard()
    {
        return view('admin.dashboard');
    }

    public function login()
    {
        return view('admin.login');
    }



    public function login_submit(Request $request)
    {
        $request->validate([
            'email' => 'required | email',
            'password' => 'required',
        ]);


        $check = $request->all();

        $data = [
            'email' => $check['email'],
            'password' => $check['password'],
        ];

        if (Auth::guard('admin')->attempt($data)) {
            return redirect()->route('admin_dashboard')->with('success', 'Login Success');
        } else {
            return redirect()->route('admin_login')->with('error', 'invalid Credentails');
        }

        return redirect()->route('admin_dashboard')->with('error', 'invalid Credentails');
    }

    public function logout()
    {
        Auth::guard('admin')->logout();

        return redirect()->route('admin_login')->with('success', 'logout Success');
    }

    public function ForgetPassword()
    {

        return view('admin.forget-password');
    }

    public function admin_forget_password_post(Request $request)
    {
        $request->validate([
            'email' => 'required | email',
        ]);

        $admin_data  = Admin::where('email', $request->email)->first();
        if (!$admin_data) {
            return redirect()->back()->with('error', 'Email Not Found');
        }

        $token = hash('sha256', time());
        $admin_data->token = $token;
        $admin_data->update();

        $reset_link = url('admin/reset-password/' . $token . '/' . $request->email);
        $subject = "Reset Password";
        $message = "please Click on below link to reset your passsword<br>";
        $message .= "<a href='" . $reset_link . "'>Click Here</a>";

        Mail::to($request->email)->send(new WebsiteMail($subject, $message));

        return redirect()->back()->with('success', 'Reset password Link send on your Email Address');
    }

    public function reset_password(Request $request, $token, $email)
    {
        $admin_data = Admin::where('email', $request->email)->where('token', $token)->first();

        if (!$admin_data) {
            return redirect()->route('admin_login')->with('error', 'Invalid token or email');
        }

        return view('admin.reset-password', compact('token', 'email'));
    }

    public function admin_reset_password_post(Request $request)
    {
        $request->validate([
            'password' => 'required',
            'password' => 'required|confirmed',
        ]);

        $admin_data = Admin::where('email', $request->email)->where('token', $request->token)->first();

        $admin_data->password = Hash::make($request->password);
        $admin_data->token = "";
        $admin_data->update();

        return redirect()->route('admin_login')->with('success', 'Password reset Successfuly');
    }
}
```

#### Mail Code
```bash
php artisan make:mail WebsiteMail
```

* WebsiteMail Class Code
```bash
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class WebsiteMail extends Mailable
{
    use Queueable, SerializesModels;

    public $subject, $body;


    public function __construct($subject, $body)
    {
        $this->subject = $subject;
        $this->body = $body;
    }

    /**
     * Get the message envelope.
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: $this->subject,
        );
    }

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'email',
        );
    }

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [];
    }
}
```

* view/email.blade.php
```bash
{{!!$body!!}}
```


