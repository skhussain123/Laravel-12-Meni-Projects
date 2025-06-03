

### Step-by-Step: Laravel 12 Custom Auth

composer create-project laravel/laravel custom-login

```bash
DB_DATABASE=your_db_name
DB_USERNAME=your_username
DB_PASSWORD=your_password

```

###  Step 1: Create Users Table
```bash
php artisan migrate
```

### Step 2: Auth Routes Setup (routes/web.php)
```bash
use App\Http\Controllers\AuthController;

Route::get('/register', [AuthController::class, 'showRegister'])->name('register');
Route::post('/register', [AuthController::class, 'register']);
Route::get('/login', [AuthController::class, 'showLogin'])->name('login');
Route::post('/login', [AuthController::class, 'login']);
Route::get('/dashboard', [AuthController::class, 'dashboard'])->middleware('auth')->name('dashboard');
Route::post('/logout', [AuthController::class, 'logout'])->name('logout');


```


### Step 3: Create AuthController
```bash
php artisan make:controller AuthController
```
**app/Http/Controllers/AuthController.php:**
```bash
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function showRegister()
    {
        return view('auth.register');
    }

    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:6|confirmed'
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        Auth::login($user);
        return redirect()->route('dashboard');
    }

    public function showLogin()
    {
        return view('auth.login');
    }

   public function login(Request $request)
{
    $credentials = $request->only('email', 'password');

    if (Auth::attempt($credentials)) {
        $user = Auth::user();

        if ($user->role === 'admin') {
            return redirect()->route('admin.panel');
        }

        return redirect()->route('dashboard');
    }

    return back()->withErrors(['email' => 'Invalid credentials']);
}




    public function dashboard()
    {
        return view('dashboard');
    }

    public function logout()
    {
        Auth::logout();
        return redirect()->route('login');
    }
}

```

### Update User Migrations
```bash
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('role')->default('user');
    });
}

```

### Step 4: Views 
**resources/views/auth/register.blade.php**
```bash
<form action="{{ url('/register') }}" method="POST">
    @csrf
    <input type="text" name="name" placeholder="Name" required><br>
    <input type="email" name="email" placeholder="Email" required><br>
    <input type="password" name="password" placeholder="Password" required><br>
    <input type="password" name="password_confirmation" placeholder="Confirm Password" required><br>
    <button type="submit">Register</button>
</form>

```

**resources/views/auth/login.blade.php**
```bash
<form action="{{ url('/login') }}" method="POST">
    @csrf
    <input type="email" name="email" placeholder="Email" required><br>
    <input type="password" name="password" placeholder="Password" required><br>
    <button type="submit">Login</button>
</form>

```

**resources/views/dashboard.blade.php**
```bash
<h1>Welcome, {{ auth()->user()->name }}</h1>
<form method="POST" action="{{ route('logout') }}">
    @csrf
    <button type="submit">Logout</button>
</form>
```

### Step 5: Middleware Setup
Laravel me auth middleware already hota hai. Aapne dashboard route me ->middleware('auth') lagaya hai, toh ye already protected hai.


