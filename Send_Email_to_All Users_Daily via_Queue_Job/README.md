
### Step-by-step Guide: Send Email to All Users Daily via Queue Job

```bash
QUEUE_CONNECTION=database

MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=xxxxxxxxxxxx
MAIL_PASSWORD=xxxxxxxxxxxxxxxx
MAIL_FROM_ADDRESS=xxxxxxxxxxxxx
MAIL_FROM_NAME="${APP_NAME}"
```

### Create Jobs table
```bash
php artisan queue:table
```

```bash
php artisan schedule:work
php artisan queue:work
```


```bash
php artisan make:job SendDailyUserEmails
```

#### app\Jobs\SendDailyUserEmails.php
```bash
<?php

namespace App\Jobs;

use App\Mail\WelcomeEmail;
use App\Models\User;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Mail;

class SendDailyUserEmails implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        //
    }

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // Get all users
        $users = User::all();

        foreach ($users as $user) {
            // Send welcome email to each user
            Mail::to($user->email)->send(new WelcomeEmail($user));
        }
    }
}

```

### Create Mail Class

```bash
php artisan make:mail DailySendEmail
```

#### Mail/app\Mail\DailySendEmail.php
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

class DailySendEmail extends Mailable
{
    use Queueable, SerializesModels;


    public $user;

    /**
     * Create a new message instance.
     */
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
            subject: 'Daily Send Email',
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

#### Boostrap/app.php
```bash
<?php

use App\Jobs\SendDailyUserEmails;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__ . '/../routes/web.php',
        commands: __DIR__ . '/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        //
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();

$app->withSchedule(function (Schedule $schedule) {
    $schedule->job(new SendDailyUserEmails())->daily();
});

```

#### routes\console.php
```bash
<?php

use Illuminate\Support\Facades\Schedule;
use App\Jobs\SendDailyUserEmails;

Schedule::job(new SendDailyUserEmails)
    ->everyTenSeconds();
```    


#### resources\views\email.blade.php
```bash
<!DOCTYPE html>
<html>
<head>
    <title>Daily Update</title>
</head>
<body>
    <h1>Hello {{ $user->name }}!</h1>
    <p>This is your daily email from our system.</p>
    <p>We hope you have a productive day!</p>
</body>
</html>

```
