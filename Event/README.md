

```bash
php artisan make:event UserRegistered
php artisan make:listener SendWelcomeEmail --event=UserRegistered
php artisan make:mail WelcomeMail
php artisan make:provider EventServiceProvider
php artisan make:Controller HomeController


```

###  File Locations

```bash
app/Events/UserRegistered.php
app/Listeners/SendWelcomeEmail.php
app/Mail/WelcomeMail.php
app/Http/Controllers/HomeController.php
app/Providers/EventServiceProvider
```

```bash
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::post('/register', [HomeController::class, 'Register'])->name('register');
```


### 1. Event: UserRegistered (Event)
```bash
public $user;

    public function __construct(User $user)
    {
          $this->user = $user;
    }
```

### 2. SendWelcomeEmail (Listeners)
```bash
<?php

namespace App\Listeners;

use App\Events\UserRegistered;
use App\Mail\WelcomeMail;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail
{
    /**
     * Create the event listener.
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     */
    public function handle(UserRegistered $event): void
    {
         Mail::to($event->user->email)->send(new WelcomeMail($event->user));
    }
}

```

### 3. EventServiceProvider (Provider)
```bash
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
use App\Events\UserRegistered;
use App\Listeners\SendWelcomeEmail;

class EventServiceProvider extends ServiceProvider
{
    /**
     * The event to listener mappings for the application.
     *
     * @var array<class-string, array<int, class-string>>
     */
    protected $listen = [
        UserRegistered::class => [
            SendWelcomeEmail::class,
        ],
    ];

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        //
    }
}

```


### 4. HomeController
```bash
function index()
    {

        return view('welcome');
    }


    function Register()
    {
        $user = User::create([
            'name' => request('name'),
            'email' => request('email'),
            'password' => bcrypt(request('password')),
        ]);

        event(new UserRegistered($user));
    }
```    


### 5. Mail Class
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

class WelcomeMail extends Mailable
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
            subject: 'Welcome Mail',
        );
    }

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'email',
            with: ['user' => $this->user],
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


### 6. views/email.blade.php
```bash
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome, {{ $user->name }}!</h1>
    <p>Thanks for joining our platform. We’re excited to have you!</p>
</body>
</html>

```

### 7. welcome.blade.php
```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>

    {{-- //user register from  --}}


    <form action="{{ route('register') }}" method="POST">
        @csrf
        <input type="name" placeholder="Name" value="hussain" name="name" id="">
        <input type="email" placeholder="Email" value="hk0527075@gmail.com" name="email" id="">
        <input type="password" placeholder="Password" value="hussain1234" name="password" id="">
        <input type="submit" value="register">

    </form>

</body>

</html>

```