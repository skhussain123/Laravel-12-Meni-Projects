


### set configartion in .env file

https://www.paypal.com/mep/dashboard

**Install first**
```bash
composer require srmklive/paypal
```

```bash
PAYPAL_SANDBOX_CLIENT_ID=AeGkN58VAwz2gIv7BpvCYlpJrJNf1xaeYqGxZJD7eQSg2oUp3r7tosw7cfMc5RILZUriqXNoFX8CiPGM
PAYPAL_SANDBOX_CLIENT_SECRET=EB9UmrX3-mYdYw1SRGzzohXmprVXTwCxKjCm9X9YnPdRpIDukVsY9nTWoy1M6aWkY0BBh2rSC4gTx3jK
PAYPAL_SANDBOX_APP_ID=sandbox
```

### Routes 
```bash
Route::get('paypal', [PayPalController::class, 'index'])->name('paypal');
Route::post('paypal/payment', [PayPalController::class, 'payment'])->name('paypal.payment');
Route::get('paypal/payment-success', [PayPalController::class, 'paymentSuccess'])->name('paypal.payment.success');
Route::get('paypal/payment-cancel', [PayPalController::class, 'paymentCancel'])->name('paypal.payment.cancel');
```



### PayPal Controller 

```bash
use Srmklive\PayPal\Services\PayPal as PayPalClient;


 public function payment(Request $request)
    {
        // Validate the amount
        $request->validate([
            'amount' => 'required|numeric|min:0.01',
        ]);

        $provider = new PayPalClient;
        $provider->setApiCredentials(config('paypal'));
        $paypalToken = $provider->getAccessToken();

        $amount = $request->amount; // Fetch the amount from the form input

        $response = $provider->createOrder([
            "intent" => "CAPTURE",
            "application_context" => [
                "return_url" => route('paypal.payment.success'),
                "cancel_url" => route('paypal.payment.cancel'),
            ],
            "purchase_units" => [
                0 => [
                    "amount" => [
                        "currency_code" => "USD",
                        "value" => $amount, // Use the dynamic amount
                    ]
                ]
            ]
        ]);

        if (isset($response['id']) && $response['id'] != null) {
            foreach ($response['links'] as $links) {
                if ($links['rel'] == 'approve') {
                    return redirect()->away($links['href']);
                }
            }

            return redirect()
                ->route('paypal')
                ->with('error', 'Something went wrong.');
        } else {
            return redirect()
                ->route('paypal')
                ->with('error', $response['message'] ?? 'Something went wrong.');
        }
    }

    public function paymentCancel()
    {
        return redirect()
            ->route('paypal')
            ->with('error', 'You have canceled the transaction.');
    }

    public function paymentSuccess(Request $request)
    {
        $provider = new PayPalClient;
        $provider->setApiCredentials(config('paypal'));
        $provider->getAccessToken();

        $response = $provider->capturePaymentOrder($request->query('token'));

        if (isset($response['status']) && $response['status'] == 'COMPLETED') {
            return redirect()
                ->route('paypal')
                ->with('success', 'Transaction complete.');
        } else {
            return redirect()
                ->route('paypal')
                ->with('error', $response['message'] ?? 'Something went wrong.');
        }
    }


```

### Payment Form
**views/payment.blade.php**

```bash
    <!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>PayPal Payment</title>
</head>

<body>
    <h1>Pay with PayPal</h1>

    @if (session('success'))
        <p style="color: green;">{{ session('success') }}</p>
    @endif

    @if (session('error'))
        <p style="color: red;">{{ session('error') }}</p>
    @endif

    <form action="{{ route('paypal.payment') }}" method="POST">
        @csrf
        <label for="amount">Enter Amount (USD):</label>
        <input type="number" name="amount" id="amount" required step="0.01" min="0.01"
            placeholder="Enter amount">
        <button type="submit">Pay Now</button>
    </form>
</body>

</html>

```


### Payment Form
**views/auth/paypal.blade.php**

```bash
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Laravel PayPal Payment Gateway Integration Example - ItSolutionStuff.com</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container">
    <div class="row mt-5 mb-5">
        <div class="col-10 offset-1 mt-5">
            <div class="card">
                <div class="card-header bg-primary">
                    <h3 class="text-white">Laravel PayPal Payment Gateway Integration Example - ItSolutionStuff.com</h3>
                </div>
                <div class="card-body">
  
                    @if ($message = Session::get('success'))
                      <div class="alert alert-success alert-dismissible fade show" role="alert">
                        <strong>{{ $message }}</strong>
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                      </div>
                    @endif
  
                    @if ($message = Session::get('error'))
                        <div class="alert alert-danger alert-dismissible fade show" role="alert">
                          <strong>{{ $message }}</strong>
                          <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                        </div>
                    @endif
                          
                    <center>
                        <a href="{{ route('paypal.payment') }}" class="btn btn-success">Pay with PayPal </a>
                    </center>
  
                </div>
            </div>
        </div>
    </div>
</div>
</body>
</html>

```



