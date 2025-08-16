

* https://github.com/Plisio/plisio-sdk-laravel
* Create account kry. account register krty howy apny wallet key mangy ge so already created wallet ki key use kro.
* goto Dashboard or My wallets pr curency Add krle


### 1. Copy Api gogo api

* Set app name ideapromptly
* Set url https://ideapromptly.com
* Copy Secret key

![Alt text](ss.png)


##### then Click See Profile and set store name url and profile
![Alt text](ss2.png)



### 2. Run Commands
```bash
composer require plisio/plisio-sdk-laravel
php artisan vendor:publish --provider="Plisio\PlisioSdkLaravel\Providers\PlisioProvider"
```

### 3. Enter Key in config/plisio.php
```bash
return [
    'api_key' => 'you_secret_key',
];
```

### 4. routes/web.php 
```bash
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PaymentController;

Route::get('/payment/form', function () {
    return view('payment.form');
})->name('payment.form');

Route::post('/payment/process', [PaymentController::class, 'processPayment'])->name('payment.process');

Route::post('/payment/callback', [PaymentController::class, 'handleCallback'])->name('payment.callback');

Route::get('/payment/success', [PaymentController::class, 'handleSuccess'])->name('payment.success');

Route::get('/payment/fail', [PaymentController::class, 'handleFailure'])->name('payment.fail');
```

### 5. resources/views/payment  (form.blade.php,success.blade.php,fail.blade.php)
##### form.blade.php
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Crypto Payment Form - IdeaPromptly</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4">
        <h1>Crypto Payment Form</h1>
        @if ($errors->any())
            <div class="alert alert-danger">
                <ul>
                    @foreach ($errors->all() as $error)
                        <li>{{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif
        <form action="{{ route('payment.process') }}" method="POST">
            @csrf
            <div class="mb-3">
                <label for="order_name" class="form-label">Order Name:</label>
                <input type="text" id="order_name" name="order_name" class="form-control" required>
            </div>
            <div class="mb-3">
                <label for="order_number" class="form-label">Order Number (Unique ID):</label>
                <input type="text" id="order_number" name="order_number" class="form-control" required>
            </div>
            <div class="mb-3">
                <label for="amount" class="form-label">Amount (in USD):</label>
                <input type="number" id="amount" name="amount" class="form-control" step="0.01" required>
            </div>
            <div class="mb-3">
                <label for="email" class="form-label">Your Email:</label>
                <input type="email" id="email" name="email" class="form-control" required>
            </div>
            <div class="mb-3">
                <label for="description" class="form-label">Description:</label>
                <textarea id="description" name="description" class="form-control"></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Pay with Crypto</button>
        </form>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>

```

##### success.blade.php
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Payment Successful - IdeaPromptly</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4 text-center">
        <h1>Payment Successful!</h1>
        <p>Order Number: {{ request()->query('order') }}</p>
        <p>Thanks for your payment!</p>
        <a href="/" class="btn btn-primary mt-3">Go Home</a>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

##### fail.blade.php
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Payment Failed - IdeaPromptly</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4 text-center">
        <h1>Payment Failed</h1>
        <p>Order Number: {{ request()->query('order') }}</p>
        <p>Please try again or contact support.</p>
        <a href="/payment/form" class="btn btn-primary mt-3">Try Again</a>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 6. app/Http/Controllers/PaymentController.php
```bash
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Plisio\PlisioSdkLaravel\Payment;

class PaymentController extends Controller
{
    public function processPayment(Request $request)
    {
        $validated = $request->validate([
            'order_name' => 'required|string',
            'order_number' => 'required|string',
            'amount' => 'required|numeric|min:0.01',
            'email' => 'required|email',
            'description' => 'nullable|string',
        ]);

        $plisioGateway = new Payment(config('plisio.api_key'));

        $data = [
            'order_name' => $validated['order_name'],
            'order_number' => $validated['order_number'],
            'source_amount' => number_format($validated['amount'], 8, '.', ''),
            'source_currency' => 'USD',
            'description' => $validated['description'] ?? 'No description',
            'email' => $validated['email'],
            'callback_url' => 'https://ideapromptly.com/payment/callback',
            'success_url' => 'https://ideapromptly.com/payment/success?order=' . $validated['order_number'],
            'cancel_url' => 'https://ideapromptly.com/payment/fail?order=' . $validated['order_number'],
            'plugin' => 'laravelSdk',
            'version' => '1.0.0',
        ];

        $response = $plisioGateway->createTransaction($data);

        if ($response && $response['status'] === 'success' && !empty($response['data'])) {
            return redirect($response['data']['invoice_url']);
        } else {
            $errorMessage = $response['data']['message'] ?? 'Error creating invoice';
            return redirect()->route('payment.form')->withErrors($errorMessage);
        }
    }

    public function handleCallback(Request $request)
    {
        $plisioGateway = new Payment(config('plisio.api_key'));
        $callbackData = $request->all();

        if ($plisioGateway->verifyCallbackData($callbackData)) {
            return response()->json(['status' => 'success']);
        } else {
            return response()->json(['error' => 'Invalid callback'], 403);
        }
    }

    public function handleSuccess(Request $request)
    {
        return view('payment.success');
    }

    public function handleFailure(Request $request)
    {
        return view('payment.fail');
    }
}
```

### 7. Cache Clear
```bash
php artisan config:cache
php artisan route:cache
php artisan view:clear
```

### 8. Test Payment 
you can use this 
* https://ideapromptly.com/payment/form






