
#### env 
```bash
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=hussainkhan2042@gmail.com
MAIL_PASSWORD=nunwwjxkzgovuogh
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=hussainkhan2042@gmail.com
MAIL_FROM_NAME="Your Name"
```



```bash
php artisan make:mail TestMail
```

#### **Mail Class**
```bash
   public function __construct($details)
    {
        $this->details = $details; // Pass email details like title and body
    }
```

```bash
  public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Test Mail from Laravel', // Set the subject dynamically or statically
        );
    }
```

```bash
 public function content(): Content
    {
        return new Content(
            view: 'emails.test', // Make sure the view is correct (resources/views/emails/test.blade.php)
            with: [
                'details' => $this->details, // Pass the email data to the view
            ]
        );
    }
```

#### **Controller Class**
```bash
php artisan make:Controller EmailController 
```

```bash
public function sendEmail()
    {
        $details = [
            'title' => 'Laravel Test Email',
            'body' => 'This is a test email sent from Laravel.'
        ];

        // Send the email
        Mail::to('hk0527075@gmail.com')->send(new TestMail($details));

        return 'Email has been sent successfully!';
    }
```    

**email/test.blade.php**

```bash
<h1>{{ $details['title'] }}</h1>
<p>{{ $details['body'] }}</p>
```






