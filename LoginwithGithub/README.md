

create api 
goto github profile -->setting--> developer setting -->auth app
create auth app web-> create api & secret key and set redirect url

 

composer require laravel/socialite

--------------------------------------------------
create github api keys
https://github.com/settings/applications/new

----------------------------------------------
http://127.0.0.1:8000/    home url
http://127.0.0.1:8000/github/callback     redirect url

--------------------------------------------------
client      Ov23liiwV1nNGVvJKox0
secret      2194cd97ab6322068116082ad219b0ebba3d1e95

-------------------------------------------------
composer require laravel/socialite

------------------------------------
set .env file 
GITHUB_CLIENT_ID=Ov23liiwV1nNGVvJKox0
GITHUB_CLIENT_SECRET=2194cd97ab6322068116082ad219b0ebba3d1e95
GITHUB_REDIRECT_URI=http://127.0.0.1:8000/github/callback

-----------------------------------

config/service.php

'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => env('GITHUB_REDIRECT_URI'),
],


-----------------------------------------

Route::get('login/github', [HomeController::class, 'redirectToProvider'])->name('github.login');
Route::get('login/github/callback', [HomeController::class, 'handleProviderCallback']);



-------------------------------------------

<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Laravel\Socialite\Facades\Socialite;

class LoginController extends Controller
{


    public function redirectToProvider()
    {
        return Socialite::driver('github')->redirect();
    }

    
 public function redirectToProvider()
    {
        return Socialite::driver('github')->redirect();
    }


    public function handleProviderCallback()
    {
        $githubUser = Socialite::driver('github')->user();


        // Find or create the user
        $user = User::updateOrCreate(
            [
                'name' => $githubUser->getName() ?? $githubUser->getNickname(),
                'email' => $githubUser->getEmail(),
                'github_id' => $githubUser->getId(),
                'password' => bcrypt(str()->random(24)),



            ]
        );


        return redirect()->intended('/'); // Redirect to the homepage or desired page
    }
}






------------------------------------------------------------












