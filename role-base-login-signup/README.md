

### Create Projects 


## Step 2: Install Laravel Breeze (for auth)
```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run dev
php artisan migrate

```


## Step 3: Set Config
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