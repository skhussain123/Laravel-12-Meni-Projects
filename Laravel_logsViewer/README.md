


https://github.com/opcodesio/log-viewer

#### Installation:
```bash
composer require opcodesio/log-viewer
```

* After installing the package, publish the front-end assets by running:
```bash
php artisan log-viewer:publish
```

#### you can see logs
http://127.0.0.1:8000/log-viewer

#### .env
```bash
LOG_CHANNEL=daily
```
* Laravel logs ko ek channel ke zariye handle karta hai. Jab aap daily channel choose karte ho, to:
* Laravel har din nayi ek log file banata hai.
* Ye files storage/logs folder mein hoti hain.
* Har roz ki alag file hone ki wajah se logs manage karna easy ho jata hai.