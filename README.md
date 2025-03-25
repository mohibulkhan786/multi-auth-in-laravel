<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

## Multi-Authentication in Laravel

**Multi-Authentication in Laravel allows you to manage multiple user roles with different access levels efficiently. Whether you need separate authentication for Admins, Employers, Job Seekers, or any other user types, Laravel provides a powerful and flexible authentication system.**

**Key Features:**
- ✅ Authentication for multiple user roles (Admin, Manager, User, etc.)
- ✅ Secure login & registration system
- ✅ Middleware-based access control
- ✅ Role-based redirection after login
- ✅ Guard-based authentication with Laravel’s built-in auth system
- ✅ Easy-to-configure and scalable authentication structure


**With Multi-Auth in Laravel, you can build secure, robust, and scalable applications with role-specific permissions, ensuring a smooth user experience.**

- ✅ Install Laravel new application

````
composer create-project "laravel/laravel:^11.0" example-app
````
- ✅ In second step, we will make database configuration for example database name, username, password etc for our crud application of laravel. So let's open <b>.env file </b>and fill all details like as:

````
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database name
DB_USERNAME=username(root)
DB_PASSWORD=password OR blank
````

- ✅ Now, we need to add new row "type" in users table and model. than we need to run migration. so let's change that on both file.
- ✅ database\migrations\0001_01_01_000000_create_users_table.php

````
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->tinyInteger('type')->default(0)->comment('Users: 0=>User, 1=>Admin, 2=>Manager');
           
            $table->rememberToken();
            $table->timestamps();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
````

- ✅ And also update User Model code <b>app\Models\User.php</b>

````
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Database\Eloquent\Casts\Attribute;

class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'type'
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

    /**

     * Interact with the user's first name.
     * @param  string  $value
     * @return \Illuminate\Database\Eloquent\Casts\Attribute
     */

    protected function type(): Attribute

    {
        return new Attribute(
            get: fn ($value) =>  ["user", "admin", "manager"][$value],
        );

    }



}

````


- ✅ Now we need to run migration. so let's run bellow command:

````
php artisan migrate
````

- ✅ Now, in this step, we will create auth using command to create login, register and dashboard. so run following commands:
- ✅ Laravel UI Packages

````
composer require laravel/ui 
````
- ✅ Generate Auth

````
php artisan ui bootstrap --auth
````
````
npm install
````

````
npm run build
````

- ✅ In this step, we require to create user access middleware that will restrict users to access that page. so let's create and update code

````
php artisan make:middleware UserAccess
````
- ✅ app/Http/middleware/UserAccess.php

````
<?php
  
namespace App\Http\Middleware;
  
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class UserAccess
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, $userType): Response
    {
        if(auth()->user()->type == $userType){
            return $next($request);
        }          
        return response()->json(['You do not have permission to access for this page.']);
    }
}

````

- ✅ Next, we need to register the UserAccess middleware to the </b>bootstrap/app.php</b> file.

````
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        
        $middleware->alias([
            'user-access' => \App\Http\Middleware\UserAccess::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();

````

- ✅ Now we will create HomeController through artisan cammnad:

````
php artisan make:controller HomeController
````

- ✅ Here, we need add adminHome() and managerHome method for admin route in HomeController. so let's add like as: <b>app/Http/Controllers/HomeController.php</b>

````
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class HomeController extends Controller
{
    //
    public function __construct()
    {
        $this->middleware('auth');

    }

    /* Show the application dashboard.
     * @return \Illuminate\Contracts\Support\Renderable
     */

    public function index(): View

    {    return view('home');

    }   

    /* Show the application dashboard.
     * @return \Illuminate\Contracts\Support\Renderable
     */

    public function adminHome(): View

    {    return view('adminHome');

    }  

    /* Show the application dashboard.
     * @return \Illuminate\Contracts\Support\Renderable
     */

    public function managerHome(): View

    {   return view('managerHome');

    }
}
````

- ✅ Here, We will add following routes group where you can create new routes for users, admins and manager access. let's update code: <b>routes/web.php</b>

````
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\HomeController;

Route::get('/', function () {
    return view('welcome');
});

Auth::routes();

/*------------------------------------------
All Normal Users Routes List
--------------------------------------------*/

Route::middleware(['auth', 'user-access:user'])->group(function () {  

    Route::get('/home', [HomeController::class, 'index'])->name('home');

});  

/*------------------------------------------
All Admin Routes List
--------------------------------------------*/

Route::middleware(['auth', 'user-access:admin'])->group(function () {  

    Route::get('/admin/home', [HomeController::class, 'adminHome'])->name('admin.home');

});  

/*------------------------------------------
All Admin Routes List
--------------------------------------------*/

Route::middleware(['auth', 'user-access:manager'])->group(function () {  

    Route::get('/manager/home', [HomeController::class, 'managerHome'])->name('manager.home');

});

````

- ✅ In this step, we need to create new blade file for admin and update for user blade file. so let's change it.<b>resources/views/home.blade.php</b>

````
@extends('layouts.app')
@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">{{ __('Dashboard') }}</div>
                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-success" role="alert">
                            {{ session('status') }}

                        </div>
                    @endif

                    You are a User.

                </div>
            </div>
        </div>
    </div>
</div>
@endsection
````

- ✅  <b>resources/views/adminHome.blade.php</b>

````
@extends('layouts.app') 

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">{{ __('Dashboard') }}</div>
                <div class="card-body">

                    You are a Admin User.

                </div>
            </div>
        </div>
    </div>
</div>
@endsection
````

- ✅  <b>resources/views/managerHome.blade.php</b>

````
@extends('layouts.app')
@section('content')

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">{{ __('Dashboard') }}</div>
                <div class="card-body">

                    You are a Manager User.

                </div>
            </div>
        </div>
    </div>
</div>
@endsection
`````

- ✅ In this step, we will change on LoginController, when user will login than we redirect according to user access. if normal user than we will redirect to home route and if admin user than we redirect to admin route. so let's change.<b>app/Http/Controllers/Auth/LoginController.php</b>

````
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;

class LoginController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Login Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles authenticating users for the application and
    | redirecting them to your home screen. The controller uses a trait
    | to conveniently provide its functionality to your applications.
    |
    */

    use AuthenticatesUsers;

    /**
     * Where to redirect users after login.
     * @var string
     */
    protected $redirectTo = '/home';

    /**
     * Create a new controller instance.
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }

    /**
     * Create a new controller instance.
     * @return RedirectResponse
     */
    public function login(Request $request): RedirectResponse
    {   
        $input = $request->all();
     
        $this->validate($request, [
            'email' => 'required|email',
            'password' => 'required',
        ]);
     
        if(auth()->attempt(array('email' => $input['email'], 'password' => $input['password'])))
        {
            if (auth()->user()->type == 'admin') {
                return redirect()->route('admin.home');
            }else if (auth()->user()->type == 'manager') {
                return redirect()->route('manager.home');
            }else{
                return redirect()->route('home');
            }
        }else{
            return redirect()->route('login')
                ->with('error','Email-Address And Password Are Wrong.');
        }
          
    }
}

````
- ✅ We will create seeder for create new admin and normal user. so let's create seeder using following command:

````
php artisan make:seeder CreateUsersSeeder
`````
- ✅  <b>database/seeders/CreateUsersSeeder.php</b>


````
<?php

namespace Database\Seeders;  

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use App\Models\User;  

class CreateUsersSeeder extends Seeder

{
    /**
     * Run the database seeds.
     * @return void
     */

    public function run(): void

    {
        $users = [
            [
               'name'=>'Admin User',
               'email'=>'admin@gmail.com',
               'type'=>1,
               'password'=> bcrypt('123456'),

            ],

            [
               'name'=>'Manager User',
               'email'=>'manager@gmail.com',
               'type'=> 2,
               'password'=> bcrypt('123456'),

            ],

            [
               'name'=>'User',
               'email'=>'user@gmail.com',
               'type'=>0,
               'password'=> bcrypt('123456'),

            ],

        ];

    

        foreach ($users as $key => $user) {

            User::create($user);

        }

    }
}
    ````

 - ✅  Now let's run seeder htrough php artisan command:

 ````
 php artisan db:seed --class=CreateUsersSeeder
 ````

 - ✅ All the required steps have been done, now you have to type the given below command and hit enter to run the Laravel app:

````
 php artisan serve
 ````
