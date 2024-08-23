# FilamentPHP - OnlineActivityUsers 
The source code for the [Filament](https://filamentphp.com) website.
# Laravel - OnlineActivityUsers 
The source code for the [Laravel](https://laravel.com/) website.

![alt text](https://github.com/Hitsukaya/FilamentPHP-Likes-Post-Blog/blob/main/blog%20post%20like%20button%20filamentphp.png "Like Button")


## Developer: Hitsukaya Valentaizar
## Email:valentaizarstone@gmail.com / contact@hitsukaya.com 

# Install - Laravel & LiveWire & FilamentPHP & TailwindCSS & JetStream

1. Create migration
```
php artisan make:migration create_users_table.php
```
## Add line in miggration file - $table->timestamp('online_at')->nullable();

        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('online_at')->nullable();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->foreignId('current_team_id')->nullable();
            $table->string('profile_photo_path', 2048)->nullable();
            $table->timestamps();
        });
```

```
2. Add line in user model
```
add line 'online_at'
```

```
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
        'online_at',
    ];

```
3. Add line in user model

```
add line  'online_at' => 'datetime',
```
```
    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
        'online_at' => 'datetime',
    ];
 
```
4. Create Middleware - OnlineMiddleware
```
 php artisan make:middleware OnlineMiddleware
```

```
 <?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpFoundation\Response;
use Exception;

class OnlineMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function handle(Request $request, Closure $next): Response
    {
        try {
            // Checks if the user is authenticated
            if (Auth::check()) {
                // Updates the 'online_at' field for the authenticated user
                Auth::user()->update(['online_at' => now()->addMinutes(5)]);
            }
        } catch (Exception $e) {
            \Log::error('Error updating online_at: ' . $e->getMessage());
        }

        return $next($request);
    }
}


 
```

4. Add in Providers/Filament/AdminPanelProvider.php

```
  use App\Http\Middleware\OnlineMiddleware;
```

```
 ->authMiddleware([OnlineMiddleware::class,])
```

5. Create Resources User

```
php artisan make:filament-resource User --view
```
## Add line or import class
```
use Filament\Tables\Columns\IconColumn;
use Illuminate\Database\Eloquent\Model;
```
Add lines in UserResource
```
   public static function table(Table $table): Table
    {
        return $table
        ->columns([
            Tables\Columns\TextColumn::make('name')
                  ->searchable(),
            Tables\Columns\TextColumn::make('email')
                  ->searchable(),
            IconColumn::make('online_at')
                  ->label('Status')
                  ->icon('heroicon-o-wifi')
                  ->color(fn (Model $record) => $record->online_at > now() ? 'success' : 'gray')
                  ->sortable(),
        ])
            ->filters([
                //
            ])
            ->actions([
                 Tables\Actions\ViewAction::make(),
                 Tables\Actions\EditAction::make(),
                 Tables\Actions\DeleteAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }
```
6. Create Controller PublicProfile

```
php artisan make:controller PublicProfileController
```

## Add in PublicProflileController

```
<?php

namespace App\Http\Controllers;

use App\Models\User;

class PublicProfileController extends Controller
{
    public function show(User $user)
    {

        if (!$user) {
            abort(404);
        }


        return view('profile.public', ['user' => $user]);
    }
}



```


7. Create in views/profile/public.blade.php
## Add this code

```
<section class="px-2 py-4 select-none dark:bg-black">
<div class="py-8 bg-white border border-gray-200 shadow-md lg:p-8 dark:bg-black dark:rounded-md dark:border-red-700">


    <img class="w-32 h-32 mx-auto rounded-full" src="{{ $user->profile_photo_url }}" alt="{{ $user->name }}">
    <h2 class="mt-3 text-3xl font-semibold text-center dark:text-gray-200">{{ $user->name }}</h2>
    <h2 class="mt-3 text-xl font-semibold text-center dark:text-gray-200">{{ $user->email }} </h2>
    <h2 class="mt-3 text-xl font-semibold text-center dark:text-gray-200">{{ $user->about }}</h2>


    <div class="flex items-center justify-center px-6 py-2 gap-x-4 ">
        <p class="inline-flex gap-2 text-blue-700">
        <svg class="text-blue-700 w-7 h-7" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path d="M10.968 18.769C15.495 18.107 19 14.434 19 9.938a8.49 8.49 0 0 0-.216-1.912C20.718 9.178 22 11.188 22 13.475a6.1 6.1 0 0 1-1.113 3.506c.06.949.396 1.781 1.01 2.497a.43.43 0 0 1-.36.71c-1.367-.111-2.485-.426-3.354-.945A7.434 7.434 0 0 1 15 19.95a7.36 7.36 0 0 1-4.032-1.181z" fill="currentColor"/>
            <path d="M7.625 16.657c.6.142 1.228.218 1.875.218 4.142 0 7.5-3.106 7.5-6.938C17 6.107 13.642 3 9.5 3 5.358 3 2 6.106 2 9.938c0 1.946.866 3.705 2.262 4.965a4.406 4.406 0 0 1-1.045 2.29.46.46 0 0 0 .386.76c1.7-.138 3.041-.57 4.022-1.296z" fill="currentColor"/>
        </svg>
        {{ $user->comments->count() }}</p>
        <p class="inline-flex gap-2 text-red-400">
            <svg class="inline-flex w-5 h-5 text-red-500 fill-current" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                <path d="M2 9.1371C2 14 6.01943 16.5914 8.96173 18.9109C10 19.7294 11 20.5 12 20.5C13 20.5 14 19.7294 15.0383 18.9109C17.9806 16.5914 22 14 22 9.1371C22 4.27416 16.4998 0.825464 12 5.50063C7.50016 0.825464 2 4.27416 2 9.1371Z" fill="currentColor"/>
            </svg>

        {{ $user->likes->count() }}</p>
        @if($user->role == 'ADMIN' || $user->role == 'EDITOR')
        <p class="inline-flex gap-2 text-blue-700">
            <svg class="inline-flex w-5 h-5 text-blue-500 fill-current" viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                <g id="icomoon-ignore"></g>
                <path d="M9.069 2.672v14.928h-6.397c0 0 0 6.589 0 8.718s1.983 3.010 3.452 3.010c1.469 0 16.26 0 20.006 0 1.616 0 3.199-1.572 3.199-3.199 0-1.175 0-23.457 0-23.457h-20.259zM6.124 28.262c-0.664 0-2.385-0.349-2.385-1.944v-7.652h5.331v7.192c0 0.714-0.933 2.404-2.404 2.404h-0.542zM28.262 26.129c0 1.036-1.096 2.133-2.133 2.133h-17.113c0.718-0.748 1.119-1.731 1.119-2.404v-22.12h18.126v22.391z" fill="currentColor"></path>
                <path d="M12.268 5.871h13.861v1.066h-13.861v-1.066z" fill="currentColor"></path>
                <path d="M12.268 20.265h13.861v1.066h-13.861v-1.066z" fill="currentColor"></path>
                <path d="M12.268 23.997h13.861v1.066h-13.861v-1.066z" fill="currentColor"></path>
                <path d="M26.129 9.602h-13.861v7.997h13.861v-7.997zM25.063 16.533h-11.729v-5.864h11.729v5.864z" fill="currentColor"></path>
            </svg>

            {{ $user->posts->count() }} </p>
        @endif





        <div class="flex items-center space-x-2">
            <!-- Icon Wi-Fi -->
            @if($user->online_at && $user->online_at->gt(now()))
                <!-- Online (Wi-Fi Icon Green) -->
                {{-- <svg class="w-6 h-6 text-green-500 fill-current" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path d="M2 9.1371C2 14 6.01943 16.5914 8.96173 18.9109C10 19.7294 11 20.5 12 20.5C13 20.5 14 19.7294 15.0383 18.9109C17.9806 16.5914 22 14 22 9.1371C22 4.27416 16.4998 0.825464 12 5.50063C7.50016 0.825464 2 4.27416 2 9.1371Z" />
                </svg> --}}

                <svg class="w-6 h-6 text-green-500 fill-current" viewBox="0 0 288 288" xmlns="http://www.w3.org/2000/svg">
                    <path d="M144 174c-17 0-34 7-45 20-10 12-28-4-18-16 16-18 39-28 63-28s47 10 63 28c10 12-8 28-18 16-11-13-28-20-45-20zm0 84c-13 0-24-11-24-24s11-24 24-24 24 11 24 24-11 24-24 24zm0-144c-32 0-61 11-84 34-10 10-29-5-17-17 27-27 63-41 101-41s74 14 101 41c12 12-7 27-17 17-23-23-52-34-84-34zm0-60c-46 0-89 17-122 48-12 11-29-6-17-17 39-36 87-55 139-55s100 19 139 55c12 11-5 28-17 17-33-31-76-48-122-48z"/>
                </svg>
            @else
                <!-- Offline (Wi-Fi Icon Gray) -->
                {{-- <svg class="w-6 h-6 text-gray-500 fill-current" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path d="M2 9.1371C2 14 6.01943 16.5914 8.96173 18.9109C10 19.7294 11 20.5 12 20.5C13 20.5 14 19.7294 15.0383 18.9109C17.9806 16.5914 22 14 22 9.1371C22 4.27416 16.4998 0.825464 12 5.50063C7.50016 0.825464 2 4.27416 2 9.1371Z" />
                </svg> --}}

                <svg class="w-6 h-6 text-gray-500 fill-current" viewBox="0 0 288 288" xmlns="http://www.w3.org/2000/svg">
                    <path d="M144 174c-17 0-34 7-45 20-10 12-28-4-18-16 16-18 39-28 63-28s47 10 63 28c10 12-8 28-18 16-11-13-28-20-45-20zm0 84c-13 0-24-11-24-24s11-24 24-24 24 11 24 24-11 24-24 24zm0-144c-32 0-61 11-84 34-10 10-29-5-17-17 27-27 63-41 101-41s74 14 101 41c12 12-7 27-17 17-23-23-52-34-84-34zm0-60c-46 0-89 17-122 48-12 11-29-6-17-17 39-36 87-55 139-55s100 19 139 55c12 11-5 28-17 17-33-31-76-48-122-48z"/>
                </svg>
            @endif

            <!-- Display the last active time -->
            <p class="dark:text-white">
                Active: {{ $user->online_at ? $user->online_at->diffForHumans() : 'Never' }}
            </p>
        </div>


    </div>

</div>
</section>

```

8. Add lines in routes/web.php
```
    use App\Http\Controllers\PublicProfileController;
```

```
    Route::get('/{user:name}', [PublicProfileController::class, 'show'])->name('profile.public');
```

