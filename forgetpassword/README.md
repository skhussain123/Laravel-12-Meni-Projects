# Forget Password using Email

## ** Step: 1**

Yes, that is the URL that will take the user to the forget page.
```bash
Route::get('/forgot-password', [HomeController::class, 'forgetPassword'])->name('passwordForgot');
```

```bash
 function forgetPassword()
    {
        return view('forget-password');
    }
```

## ** Step: 2 forget-password.blade.php**
This is the page where the forget password form is.

```bash
  <div class="card p-4">
        @if (session('success'))
            <div class="alert alert-success" role="alert">
                {{ session('success') }}
            </div>
        @endif
        @if (session('error'))
            <div class="alert alert-danger" role="alert">
                {{ session('error') }}
            </div>
        @endif


        <h3 class="text-center mb-3">Forgot Password</h3>
        <form action="{{ route('post-forgot-password') }}" method="POST">
            @csrf
            <div class="mb-3">
                <label for="email" class="form-label">Enter your email</label>
                <input type="email" class="form-control" name="f-email" id="email"
                    placeholder="example@example.com" required>
                @error('f-email')
                    <div class="text-danger">{{ $message }}</div>
                @enderror
            </div>
            <button type="submit" class="btn btn-primary">Send Reset Link</button>
            <div class="text-center mt-3">
                <a href="login.html">Back to Login</a>
            </div>
        </form>

        
    </div>
```

## ** Step: 3**
```bash
Route::post('/post-forgot-password', [HomeController::class, 'postForgetPassword'])->name('post-forgot-password');
```


```bash
 public function postForgetPassword(Request $request)
    {
        $request->validate([
            'f-email' => 'required|email',
        ]);

        $user = User::where('email', $request->input('f-email'))->first();

        if ($user) {
            $token = Str::random(40);
            $domain = URL::to('/');
            $url = $domain . '/recover-password?token=' . $token;

            $data['url'] =  $url;
            $data['email'] = $user->email; // ✅ Fix here
            $data['title'] =  'Password Reset';
            $data['body'] =  'We received a request to reset the password associated with this email address. If you made this request, please click the button below to reset your password. If you did not request a password reset, you can safely ignore this email.';

            Mail::send('email.User.registerVerify', ['data' => $data], function ($message) use ($data) {
                $message->to($data['email'])->subject($data['title']);
            });

            $dateTime = Carbon::now()->format('Y-m-d H:i:s');

            PasswordReset::updateOrCreate(
                ['email' => $user->email],
                [
                    'email' => $user->email,
                    'token' => $token,
                    'created_at' => $dateTime,
                    'updated_at' => $dateTime,
                ]
            );

            return redirect()->back()->with('success', 'Reset password link sent to your email!');
        } else {
            return redirect()->back()->with('error', 'Email not found!');
        }
    }
    ```