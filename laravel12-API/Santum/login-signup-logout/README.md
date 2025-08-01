

### install API packages
php artisan install:api

**make automatically folder / file after installing** 
Routes/api.php

* Sanctum  --> medium level this this packages
* Passport --> advance porjects level this this packages

### Sanctum Laravel

Sanctum Laravel ka ek lightweight authentication system hai jo mainly SPA (Single Page Applications), mobile applications, aur simple token-based APIs ke liye use hota hai.

Token aik unique string hoti hai jo user ko authenticate karne ke liye use hoti hai. Jab koi user login karta hai, to Laravel Sanctum us user ke liye ek token generate karta hai jo future requests ke sath send ki jati hai taake user verify ho sake.


### Simple Test api Route
```bash
Route::get('/test', function () {
    return ['name' => "hussain", 'channel' => 'hussian khan'];
});
```

### Laravel HTTP Status Codes Table

| **Status Code**             | **Meaning**                  | **Use Case**                      | **Laravel Response Example**                                 |
| --------------------------- | ---------------------------- | --------------------------------- | ------------------------------------------------------------ |
| `200 OK`                    | Request successful           | GET request, successful login     | `return response()->json(['message' => 'Success'], 200);`    |
| `201 Created`               | Resource created             | Register, POST to create data     | `return response()->json(['message' => 'Created'], 201);`    |
| `204 No Content`            | Success but no data returned | DELETE request                    | `return response()->noContent();`                            |
| `400 Bad Request`           | Invalid input/request        | Validation error                  | `return response()->json(['error' => 'Bad Request'], 400);`  |
| `401 Unauthorized`          | Authentication failed        | Invalid token/login failed        | `return response()->json(['error' => 'Unauthorized'], 401);` |
| `403 Forbidden`             | Access denied                | User doesn’t have permission      | `return response()->json(['error' => 'Forbidden'], 403);`    |
| `404 Not Found`             | Resource not found           | Invalid route or ID               | `return response()->json(['error' => 'Not Found'], 404);`    |
| `422 Unprocessable Entity`  | Validation error             | Form validation failed            | Automatically returned by Laravel validator                  |
| `500 Internal Server Error` | Server crash/error           | Unhandled exception, server issue | `return response()->json(['error' => 'Server Error'], 500);` |



# Login / Register / Logout Api




### Create controller
```bash
php artisan make:Controller Api/AuthController
```



### AuthController
```bash

use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;


 public function signup(Request $request)
    {
        // Validation
        $validateUser = Validator::make(
            $request->all(),
            [
                'name' => 'required|string|max:255',
                'email' => 'required|email|unique:users,email',
                'password' => 'required|min:6',
            ]
        );

        if ($validateUser->fails()) {
            return response()->json([
                'status' => false,
                'message' => 'Validation Error',
                'errors' => $validateUser->errors()->all()
            ], 422); // 422 = Unprocessable Entity
        }

        // Create user
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        // Token create karein
        $token = $user->createToken("auth_token")->plainTextToken;

        return response()->json([
            'status' => true,
            'message' => 'User created successfully',
            'user' => $user,
            'token' => $token
        ], 201); // 201 = Created
    }


    public function login(Request $request)
    {
        // Validation
        $validateUser = Validator::make(
            $request->all(),
            [
                'email' => 'required|email',
                'password' => 'required|min:6',
            ]
        );

        if ($validateUser->fails()) {
            return response()->json([
                'status' => false,
                'message' => 'Validation Error',
                'errors' => $validateUser->errors()->all()
            ], 422);
        }

        // User find
        $user = User::where('email', $request->email)->first();

        // Check credentials
        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json([
                'status' => false,
                'message' => 'Invalid credentials',
            ], 401);
        }

        // Generate token
        $token = $user->createToken("auth_token")->plainTextToken;

        return response()->json([
            'status' => true,
            'message' => 'User logged in successfully',
            'user' => $user,
            'token' => $token,
            'token_type' => 'Bearer',
        ], 200);
    }


    public function logout(Request $request)
    {
        // Revoke the token that was used to authenticate the current request
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'status' => true,
            'message' => 'User logged out successfully.'
        ], 200);
    }

```

### Routes 
route/api.php

```bash
Route::post('/signup', [AuthController::class, 'signup'])->name('SignUpM');
Route::post('/login', [AuthController::class, 'login']);
Route::post('/logout', [AuthController::class, 'logout'])->name('logout')->middleware('auth:sanctum');

```