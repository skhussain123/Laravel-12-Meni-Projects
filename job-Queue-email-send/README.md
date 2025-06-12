Laravel queues background tasks (jaise emails bhejna, video processing, etc.) ko asynchronously run karne ke liye use hoti hain. Agar aap database queue driver use kar rahe hain, to aapko pehle ek jobs table banana padta hai jahan ye queued jobs store hongi.


```bash
php artisan queue:tabel
```

php artisan queue:table run karne par ek migration file create hoti hai:
database/migrations/xxxx_xx_xx_xxxxxx_create_jobs_table.php


```bash
php artisan make:job SendWelcomeEmail
```

```bash
php artisan make:mail WelcomeEmail
```
**queue Status Command**
```bash
php artisan queue:work
```

##### .env
```bash
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=xxxxxxxxxxxxxxxxxxxxx
MAIL_PASSWORD=xxxxxxxxxx
MAIL_FROM_ADDRESS=xxxxxxxxxxxxxxxxxxx
MAIL_FROM_NAME="${APP_NAME}"

```
##### Routes
```bash
Route::get('/', HomeController::class . '@callWelcome')->name('home');
Route::post('/user-register', [HomeController::class, 'userRegister'])->name('user.register');
```

##### HomeController
```bash
<?php

namespace App\Http\Controllers;

use App\Jobs\SendEmail;
use App\Jobs\SendWelcomeEmail;
use App\Mail\WelcomeEmail;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class HomeController extends Controller
{
    public function callWelcome()
    {

        return view('welcome');
    }

    function userRegister(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'email' => 'required',
            'password' => 'required|min:6',
        ]);


        //register user
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => bcrypt($request->password),
        ]);

        SendWelcomeEmail::dispatch($user);
        return redirect()->route('home')->with('success', 'User registered successfully!');
    }
}

```

##### Jobs/SendWelcomeEmail
```bash
<?php

namespace App\Jobs;

use App\Mail\WelcomeEmail;
use App\Models\User;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use Queueable;

    protected $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Mail::to($this->user->email)->send(new WelcomeEmail($this->user));
    }
}

```

##### Mail/WelcomeEmail
```bash
<?php

namespace App\Mail;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class WelcomeEmail extends Mailable
{
    use Queueable, SerializesModels;

    public $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }



    /**
     * Get the message envelope.
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Welcome Email',
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

##### views/email
```bash
<h1>Welcome, {{ $user->name }}!</h1>
<p>Thank you for registering with us.</p>
```


##### views/welcome.blade.php
```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>User Registration</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f8f9fa;
        }

        .form-container {
            max-width: 500px;
            margin: 50px auto;
            padding: 30px;
            background-color: white;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>

<body>

    <div class="container">
        <div class="form-container">

            @if (session('success'))
                <span>{{ session('success') }}</span>
            @else
            @endif
            <h2 class="text-center mb-4">User Registration</h2>
            <form action="{{ route('user.register') }}" method="POST">
                @csrf
                <div class="mb-3">
                    <label for="name" class="form-label">Full Name</label>
                    <input type="text" class="form-control" id="name" name="name"
                        placeholder="Enter your full name" required>
                </div>
                <div class="mb-3">
                    <label for="email" class="form-label">Email address</label>
                    <input type="email" class="form-control" id="email" name="email"
                        placeholder="Enter your email" required>
                </div>
                <div class="mb-3">
                    <label for="password" class="form-label">Password</label>
                    <input type="password" class="form-control" id="password" name="password"
                        placeholder="Enter your password" required>
                </div>
                <button type="submit" class="btn btn-primary w-100">Register</button>
            </form>
        </div>
    </div>

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>

</html>


```