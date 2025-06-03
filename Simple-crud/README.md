

###  Step 1: Create a Laravel 12 Project (if not already)
```bash
composer create-project laravel/laravel laravel-form-app
cd laravel-form-app

```

### Step 2: Configure Database
Open .env file and update your database credentials:

```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database_name
DB_USERNAME=your_username
DB_PASSWORD=your_password

```
```bash
php artisan config:cache
```

### Step 3: Create a Model and Migration

```bash
php artisan make:model User -m
```
**Now open the migration file in database/migrations/ and add:**

```bash
public function up(): void
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamps();
    });
}
```
```bash
php artisan migrate
```


### Step 4: Create Controller
```bash
php artisan make:controller UserController
```
**In app/Http/Controllers/UserController.php, add:**
```bash
use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function create()
    {
        return view('user-form');
    }

    public function store(Request $request)
    {
        $request->validate([
            'name'  => 'required|string',
            'email' => 'required|email|unique:users',
        ]);

        User::create([
            'name'  => $request->name,
            'email' => $request->email,
        ]);

        return redirect()->back()->with('success', 'User added successfully!');
    }
}

```

### Step 5: Add Fillable Fields in Model
In app/Models/User.php, add:

```bash
protected $fillable = ['name', 'email'];
```


### Step 6: Create a Blade View
Create a view file: resources/views/user-form.blade.php

```bash
<!DOCTYPE html>
<html>
<head>
    <title>User Form</title>
</head>
<body>
    <h1>Add New User</h1>

    @if(session('success'))
        <p style="color: green;">{{ session('success') }}</p>
    @endif

    <form action="{{ route('users.store') }}" method="POST">
        @csrf
        <label>Name:</label><br>
        <input type="text" name="name" required><br><br>

        <label>Email:</label><br>
        <input type="email" name="email" required><br><br>

        <button type="submit">Submit</button>
    </form>
</body>
</html>

```


### Step 7: Define Routes
In routes/web.php, add:

```bash
use App\Http\Controllers\UserController;

Route::get('/user-form', [UserController::class, 'create'])->name('users.create');
Route::post('/user-form', [UserController::class, 'store'])->name('users.store');

```


### Step 8: Test in Browser
```bash
php artisan serve
```
Go to:
http://localhost:8000/user-form



### Data insert With Different Methods:

#### 1. create() – (Mass Assignment)
```bash
User::create([
    'name' => $request->name,
    'email' => $request->email,
]);
```

#### 2. insert() – (For multiple records, no timestamps, direct DB insert)
```bash
User::insert([
    ['name' => 'Ali', 'email' => 'ali@example.com'],
    ['name' => 'Ahmed', 'email' => 'ahmed@example.com'],
]);
```

#### 3. save() – (After creating a model object)
```bash
$user = new User();
$user->name = $request->name;
$user->email = $request->email;
$user->save();
```