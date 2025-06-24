

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