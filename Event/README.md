

```bash
php artisan make:event UserRegistered
php artisan make:listener SendWelcomeEmail --event=UserRegistered
php artisan make:mail WelcomeMail
php artisan make:provider EventServiceProvider


```

###  File Locations

```bash
app/Events/UserRegistered.php
app/Listeners/SendWelcomeEmail.php
```


### 1. Event: UserRegistered
