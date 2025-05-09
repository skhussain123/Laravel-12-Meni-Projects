# Forget Password using Email

## **Step: 1**

Yes, that is the URL that will take the user to the forget page.
```bash
Route::get('/forgot-password', [HomeController::class, 'forgetPassword'])->name('passwordForgot');
```

```bash
 function forgetPassword()
    {
        return view('forget-password');
    }
```

## **Step: 2 forget-password.blade.php**
This is the page where the forget password form is.

```bash
  <div class="card p-4">
        @if (session('success'))
            <div class="alert alert-success" role="alert">
                {{ session('success') }}
            </div>
        @endif
        @if (session('error'))
            <div class="alert alert-danger" role="alert">
                {{ session('error') }}
            </div>
        @endif


        <h3 class="text-center mb-3">Forgot Password</h3>
        <form action="{{ route('post-forgot-password') }}" method="POST">
            @csrf
            <div class="mb-3">
                <label for="email" class="form-label">Enter your email</label>
                <input type="email" class="form-control" name="f-email" id="email"
                    placeholder="example@example.com" required>
                @error('f-email')
                    <div class="text-danger">{{ $message }}</div>
                @enderror
            </div>
            <button type="submit" class="btn btn-primary">Send Reset Link</button>
            <div class="text-center mt-3">
                <a href="login.html">Back to Login</a>
            </div>
        </form>

        
    </div>
```

## **Step: 3**
```bash
Route::post('/post-forgot-password', [HomeController::class, 'postForgetPassword'])->name('post-forgot-password');
```


```bash
use App\Mail\ExampleMail;
use App\Mail\User\RegVerifyMail;
use App\Mail\UserRegl;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Mail;
use App\Mail\VerifyMail;
use App\Models\PasswordReset;
use Carbon\Carbon;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\Str;
```

```bash
 public function postForgetPassword(Request $request)
    {
        $request->validate([
            'f-email' => 'required|email',
        ]);

        $user = User::where('email', $request->input('f-email'))->first();

        if ($user) {
            $token = Str::random(40);
            $domain = URL::to('/');
            $url = $domain . '/recover-password?token=' . $token;

            $data['url'] =  $url;
            $data['email'] = $user->email; // ✅ Fix here
            $data['title'] =  'Password Reset';
            $data['body'] =  'We received a request to reset the password associated with this email address. If you made this request, please click the button below to reset your password. If you did not request a password reset, you can safely ignore this email.';

            Mail::send('email.User.registerVerify', ['data' => $data], function ($message) use ($data) {
                $message->to($data['email'])->subject($data['title']);
            });

            $dateTime = Carbon::now()->format('Y-m-d H:i:s');

            PasswordReset::updateOrCreate(
                ['email' => $user->email],
                [
                    'email' => $user->email,
                    'token' => $token,
                    'created_at' => $dateTime,
                    'updated_at' => $dateTime,
                ]
            );

            return redirect()->back()->with('success', 'Reset password link sent to your email!');
        } else {
            return redirect()->back()->with('error', 'Email not found!');
        }
    }
```

## **Step: 4: Models**
```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class PasswordReset extends Model
{
    protected $table = 'password_resets';

    public $timestamps = false;

    protected $primaryKey = 'email';  // You're using email as primary key
    public $incrementing = false;     // Email is not incrementing
    protected $keyType = 'string';    // Email is a string

    protected $fillable = [
        'email',
        'token',
        'created_at',
        'updated_at',
    ];
}
```

```bash
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'Role'
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

## **Step: 5 Migration**
```bash
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->string('Role')->default('user');
            $table->rememberToken();
            $table->timestamps();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
```

```bash
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('password_resets', function (Blueprint $table) {
            $table->id();
            $table->string('email')->nullable();
            $table->string('token')->nullable();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('password_resets');
    }
};
```

**Step: 5 This page is bringing data from the Code Request Password Recovery Form page.**

```bash
Route::get('/recover-password', [HomeController::class, 'recoverpasswordLoad'])->name('recoverpasswordLoad');
```

```bash
    public function recoverpasswordLoad(Request $request)
    {
        // Fetch the first record matching the token
        $resetData = PasswordReset::where('token', $request->token)->first();


        if ($resetData) {
            $user = User::where('email', $resetData->email)->first();

            if ($user->id) {
                $userID = $user->id;
                return view('passwordrecover', compact('userID', 'user'));
            } else {

                return redirect()->route('account')->with('error', 'No user found with this email!');
            }
        } else {
            return redirect()->route('account')->with('error', 'token expired or invalid forget again!');
        }
    }
```

**passwordrecover.blade.php**
```bash
<div class="formbold-form-wrapper card" style="padding: 20px">
    <div class="formbold-form-title">
        <h3>Create New Password!</h3>

    </div>

    @error('error')
        <p class="text-danger">{{ $error }}</p>
    @enderror

    <form action="{{ route('recoverpassword') }}" method="post">
        @csrf

        @if ($user)
            <input type="text" name="id" value="{{ $userID }}">

            <input type="password" name="password" id="" placeholder="New password"
                class="formbold-form-input" />

            @error('password')
                <span class="text-danger">{{ $message }}</span>
            @enderror

            <br>

            <input type="password" name="password_confirmation" id="" placeholder="Confirm password"
                class="formbold-form-input mt-4" />

            @error('password_confirmation')
                <span class="text-danger">{{ $message }}</span>
            @enderror

            <br>

            <input class="formbold-btn mt-4" type="submit" value="Submit">
        @else
            <p class="text-danger">User not found or invalid token.</p>
        @endif
    </form>
</div>

<br>
```

```bash
Route::post('/reset-password', [HomeController::class, 'recoverpassword'])->name('recoverpassword');
```

```bash
 public function recoverpassword(Request $request)
    {
        $request->validate([
            'password' => 'required|string|min:6|confirmed',
            'id' => 'required',
        ]);

        try {

            $user = User::find($request->id);

            $user->password = Hash::make($request->password);
            $user->save();


            PasswordReset::where('email', $user->email)->delete();

            session(['userEmail' => $user->email]);

            return redirect()->route('account')->with('success', 'Password updated successfully!');
        } catch (\Exception $e) {
            return redirect()->back()->withErrors(['error' => $e->getMessage()]);
        }
    }
```

