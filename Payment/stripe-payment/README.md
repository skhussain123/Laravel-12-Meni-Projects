

### 1. Create Stripe Test Account and Create
https://dashboard.stripe.com/test/dashboard

#### Testing Card List
https://docs.stripe.com/testing


### 2. Create Laravel Project
```bash
composer create-project laravel/laravel stripe-app
```

```bash
composer require stripe/stripe-php
```


### 2. Create Client & Secret ID And paste .env
```bash
STRIPE_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
STRIPE_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 3. Goto the config/services.php and set this code
```bash
 'stripe' => [
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],
```  



### Create Controller
```bash
php artisan make:controller StripeController
```

### Create This Routes
```bash
Route::get('stripe', [StripeController::class, 'stripe']);
Route::post('stripe', [StripeController::class, 'stripePost'])->name('stripe.post');
```


### Create View File
views/stripe.blade.php
```bash
<!-- resources/views/stripe.blade.php -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Stripe Payment</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.6/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-4Q6Gf2aSP4eDXB8Miphtr37CMZZQ5oXLH2yaXMJ2w8e2ZtHTl7GptT4jmndRuHDT" crossorigin="anonymous">

    <style>
        .container {
            max-width: 500px;
            margin-top: 80px;
        }

        #card-element {
            padding: 10px;
            border: 1px solid #ced4da;
            border-radius: .375rem;
        }

        .StripeElement {
            background-color: white;
            padding: 10px 12px;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="card shadow">
            <div class="card-header text-center">
                <h4>Order Summary</h4>
            </div>
            <div class="card-body">

                @if (session('success'))
                    <div class="alert alert-success">
                        {{ session('success') }}
                    </div>
                @else
                @endif
                <p><strong>Product:</strong> Example Product</p>
                <p><strong>Price:</strong> $10.00</p>
                <p><strong>Quantity:</strong> 1</p>
                <p><strong>Total:</strong> $10.00</p>

                <form id="payment-form" action="{{ route('stripe.post') }}" method="POST">
                    @csrf

                    <label for="card-element" class="form-label">Credit or debit card</label>
                    <div id="card-element" class="form-control mb-3"></div>
                    <div id="card-errors" class="text-danger mb-3" role="alert"></div>

                    <input type="text" name="stripeToken" id="stripeToken">

                    <button type="submit" id="submit-button" class="btn btn-primary w-100">Pay Now</button>
                </form>
            </div>
        </div>
    </div>

    <!-- Stripe JS -->
    <script src="https://js.stripe.com/v3/"></script>

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.6/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-j1CDi7MgGQ12Z7Qab0qlWQ/Qqz24Gc6BM0thvEMVjHnfYGF0rmFCozFSxQBxwHKO" crossorigin="anonymous">
    </script>

    <script>
        const stripe = Stripe('{{ config('services.stripe.key') }}');
        const elements = stripe.elements();
        const card = elements.create('card');
        card.mount('#card-element');

        const form = document.getElementById('payment-form');
        const errorElement = document.getElementById('card-errors');

        form.addEventListener('submit', function(event) {
            event.preventDefault();

            stripe.createToken(card).then(function(result) {

                console.log(result);
                if (result.error) {
                    errorElement.textContent = result.error.message;
                } else {
                    document.getElementById('stripeToken').value = result.token.id;
                    form.submit();
                }
            });
        });
    </script>
</body>

</html>

```


### Stripe Controller
```bash
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Stripe\StripeClient;

class StripeController extends Controller
{
    public function stripe()
    {
        return view('stripe');
    }

    public function stripePost(Request $request)
    {
        $stripe = new StripeClient(config('services.stripe.secret'));

        $charge = $stripe->charges->create([
            'amount' => 1099,
            'currency' => 'usd',
            'source' => $request->stripeToken,
            'description' => 'Payment from Laravel Stripe Example',
        ]);
        
        return back()->with('success', 'Payment successful!');
    }
}


```


