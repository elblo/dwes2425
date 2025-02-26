<style>
    img { margin: 20px 0; border-radius: 8px; }

    .alert { color: #BD1550; }
    .warning { color: #E97F02; }
    .success { color: #8A9B0F; }

    .center { text-align: center; }
    .right { text-align: right; }

    .img-small { max-width: 200px; margin: auto; }
    .img-medium { max-width: 400px; margin: auto; }
    .img-large { max-width: 800px; margin: auto; }

    .leyenda {
        font-size: small;
        margin: 10px 0;
    }
</style>

# Seguridad en Laravel

> Duración estimada: 4 sesiones

## 10.1 Autenticación

### Introducción

La mayoría de aplicaciones web permiten que sus usuarios se autentiquen mediante un sistema de "Login" o "Inicio de sesión". Implementar esta función de forma manual puede ser complejo y arriesgado. Por eso, Laravel se esfuerza por proporcionar herramientas para implementar la autenticación de manera rápida, segura y fácil.

En su núcleo, las instalaciones de autenticación de Laravel se componen de "guards" (guardias) y "providers" (proveedores). Los guardias definen cómo se autentican los usuarios para cada solicitud. Por ejemplo, el guardia session mantiene el estado utilizando almacenamiento de sesión y cookies.

El archivo de configuración de autenticación de tu aplicación se encuentra en `config/auth.php`. Contiene varias opciones documentadas para ajustar el comportamiento de los servicios de autenticación de Laravel.

### Kits de inicio

Laravel proporciona unos kits de inicio que estructuran automáticamente la aplicación con las rutas, controladores y vistas necesarios para registrar y autenticar a los usuarios de la aplicación. Los más utilizados son **Breeze** y **JetStream**. Ambos, aparecen entre las opciones al crear un nuevo proyecto Laravel.

**Laravel Breeze** es una implementación simple y mínima de todas las funciones de autenticación de Laravel, que incluye inicio de sesión, registro, restablecimiento de contraseña, verificación de correo electrónico y confirmación de contraseña. La capa de vista de Laravel Breeze está compuesta por plantillas Blade con estilos de Tailwind CSS. 

**Laravel Jetstream** es un sólido kit de inicio de aplicaciones que consume y expone los servicios del backend de autenticación Laravel Fortify con una interfaz de usuario basada en Tailwind CSS, Livewire y/o Inertia. Laravel Jetstream incluye soporte opcional para la autenticación de dos factores, soporte para equipos, gestión de sesiones de navegador, gestión de perfiles e integración incorporada con Laravel Sanctum para ofrecer autenticación de token de API. 

Puedes probar a crear un proyecto y utilizar uno de estos kits, sobre todo Breeze, para navegar por sus archivos y aprender cómo funciona un sistema de autenticación.

### Sistema autenticación manual

Mejor incluso que empezar utilizando un kit de inicio, es **crear un sistema de autenticación de forma manual**. Así, aprenderás paso a paso cómo funciona la autenticación. Y es lo que vamos a hacer en este punto. Una vez que entiendas todos los puntos, para futuras aplicaciones será más práctico empezar el desarrollo con un kit de inicio.

Vamos a necesitar unas vistas con los formularios de login y registro, unas rutas y controladores con sus funciones asociadas.

#### 1. Rutas para login, registro y logout

En `web.php` agregamos las rutas necesarias para:

- Mostrar el formulario de registro y procesar sus datos.
- Mostrar el formulario de login y procesar sus datos.
- Cerrar sesión.

```php
<?php
// Rutas auth
Route::get('/register', [RegisterController::class, 'create']);  //Formulario de registro
Route::post('/register', [RegisterController::class, 'store']);  //Registrar usuario

Route::get('/login', [SessionController::class, 'create']);  //Formulario de login
Route::post('/login', [SessionController::class, 'store'])->name('login');  //Iniciar sesión

Route::post('/logout', [SessionController::class, 'destroy']);  //Cerrar sesión 
```

#### 2. Controladores necesarios

Creamos 2 controladores:

- `RegisterController`, encargado del registro de usuarios.
- `SessionController`, encargado del inicio y cierre de sesión.

```bash
php artisan make:controller RegisterController
php artisan make:controller SessionController
```

Empezamos con `RegisterController`, que va a tener 2 funciones:

- `create`: simplemente muestra la vista del formulario de registro.
- `store`: recibe los datos del formulario de registro y si son válidos, crea el usuario en la BDD, inicia su sesión y redirige a la página de inicio.

```php
<?php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rules\Password;

class RegisterController extends Controller
{
    // Muestra la vista del formulario de registro 
    public function create(){ return view('auth.register'); }

    // Procesa los datos del formulario de registro
    public function store(Request $request){
        // Validación
        $validatedAttributes = $request->validate([
            'name' => 'required',
            'email' => 'required|email',
            'password' => ['required', Password::min(8)->mixedCase()->numbers()->symbols(), 'confirmed:password_confirmation'],
        ]);

        // Creación del usuario
        $user = User::create($validatedAttributes); 

        // Inicio de sesión con la instancia del usuario (login)
        Auth::login($user);

        // Redirección a página de inicio
        return redirect('/');
    }
}
```

Es el turno de `SessionController`, que va a tener 3 funciones:

- `create`: simplemente muestra la vista del formulario de login.
- `store`: recibe los datos del formulario de login y si son válidos, intenta iniciar sesión con ellos y redirige a la página de inicio.
- `destroy`: cierra la sesión, invalidándola y redirigiendo a la página de inicio.

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

class SessionController extends Controller
{
    // Muestra la vista del formulario de login 
    public function create(){
        return view('auth.login');
    }

    // Procesa los datos del formulario de login
    public function store(Request $request){ 
       //Validar los datos email y password
       $request->validate([
            'email' => 'required|email',
            'password' => 'required',
       ]);

       $attributes = $request->only(['email', 'password']);

       // Intentar iniciar sesión con esos datos (email y password)
       if (!Auth::attempt($attributes)) {
            throw ValidationException::withMessages([
                'email' => 'Sorry, those credentials do not match.'
            ]);
        }

       // Registrar sesión regenerando su id 
       $request->session()->regenerate();

       // Redirección a la página de inicio
       return redirect('/');
    }

    // Cierra sesión
    public function destroy(Request $request){
        // Cerrar sesión
        Auth::logout();

        // Invalidar la sesión
        $request->session()->invalidate();

        // Redirección a la página de inicio
        return redirect('/');
    }
}
```

#### 3. Vistas login y register

Vamos a crear las vistas para los formularios de inicio de sesión y registro dentro de una carpeta común `auth` para su mejor organización.

Vista para el fomulario de registro en `/resources/views/auth/register.blade.php`:

```html
@extends('layout.app')
@section('content')
<div class="flex min-h-full items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
    <div class="w-full max-w-md space-y-8">
      <div>
        <h2 class="mt-6 text-center text-3xl font-bold tracking-tight text-gray-900">Registro</h2>
      </div>
      <form method="POST" action="/register">
        @csrf

        <div class="space-y-12">
          <div class="border-b border-gray-900/10 pb-12">
            <div class="flex flex-col gap-3">
              
                <div class="mb-4">
                    <label for="name" class="block text-sm font-medium text-gray-700">Nombre</label>
                    <div class="mt-2">
                        <input type="text" name="name" id="name" value="{{ old('name')}}" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('name')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>
                
                <div class="mb-4">
                    <label for="email" class="block text-sm font-medium text-gray-700">Email</label>
                    <div class="mt-2">
                        <input type="email" name="email" id="email" value="{{ old('email')}}" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('email')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>
                
                <div class="mb-4">
                    <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
                    <div class="mt-2">
                        <input type="password" name="password" id="password" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('password')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>
                
                <div class="mb-4">
                    <label for="password_confirmation" class="block text-sm font-medium text-gray-700">Repite el Password</label>
                    <div class="mt-2">
                        <input type="password" name="password_confirmation" id="password_confirmation" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('password_confirmation')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>
                

            </div>
          </div>
        </div>

        <div class="mt-6 flex items-center justify-end gap-x-6">
          <a href="/" class="text-sm font-semibold leading-6 text-gray-900">Cancelar</a>
          <input type="submit" value="Registro" />
        </div>
      </form>
    </div>
  </div>
@endsection
```

Vista para el fomulario de login en `/resources/views/auth/login.blade.php`:

```html
@extends('layout.app')
@section('content')
<div class="flex min-h-full items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
    <div class="w-full max-w-md space-y-8">

      <div>
        <h2 class="mt-6 text-center text-3xl font-bold tracking-tight text-gray-900">Log In</h2>
      </div>

      <form class="flex-[0.5]" method="POST" action="/login">
        @csrf

        <div class="space-y-12">
          <div class="border-b border-gray-900/10 pb-12">
            <div class="flex flex-col gap-3">

                <div class="mb-4">
                    <label for="email" class="block text-sm font-medium text-gray-700">Email</label>
                    <div class="mt-2">
                        <input type="email" name="email" id="email" value="{{ old('email') }}" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('email')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>
                
                <div class="mb-4">
                    <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
                    <div class="mt-2">
                        <input type="password" name="password" id="password" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('password')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>
                
            </div>
          </div>
        </div>

        <div class="mt-6 flex items-center justify-end gap-x-6">
          <a href="/" class="text-sm font-semibold leading-6 text-gray-900">Cancelar</a>
          <input type="submit" value="Log In" />
        </div>
      </form>
    </div>
  </div>
@endsection
```

#### 4. Vistas de plantilla base y menu

Crear la vista con la plantilla base en `/resources/views/layout/app.blade.php`:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>Laravel</title>

        <!-- Fonts -->
        <link rel="preconnect" href="https://fonts.bunny.net">
        <link href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap" rel="stylesheet" />

        <!-- Styles / Scripts -->
        @vite(['resources/css/app.css', 'resources/js/app.js'])
    
    </head>
    <body class="font-sans antialiased">
       <header>
            @include('partials.menu')
       </header>
       <main>
            @yield('content')
       </main>
       <footer>
            Prueba de autenticación
       </footer>
    </body>
</html>
```

Con la directiva **@auth** podemos mostrar contenido si el usuario está logueado y con **@guest** si no lo está. Realmente son un if encubierto, donde simplemente se comprueba si existe un usuairo logueado o no. En la siguiente vista los utilizamos para mostrar unos menús u otros según el estado de la sesión.

Crear la vista con el menú con la navegación y opciones en `/resources/views/partials/menu.blade.php`:

```html
<div class="flex h-16 items-center justify-between">
    <div class="flex items-center">
        <div class="ml-10 flex items-baseline space-x-4">
            
            @php
                $currentRoute = request()->path();
            @endphp

            <nav class="flex space-x-4">
                <a href="/" class="{{ $currentRoute === '/' ? 'text-indigo-600 font-semibold' : 'text-gray-600 hover:text-indigo-600' }}">
                    Inicio
                </a>
                <a href="/usuarios" class="{{ $currentRoute === 'usuarios' ? 'text-indigo-600 font-semibold' : 'text-gray-600 hover:text-indigo-600' }}">
                    Usuarios
                </a>
                <!--
                    ...
                -->
            </nav>

        </div>
    </div>

    <!-- Enlaces para login/register -->
    <div class="ml-4 flex items-center md:ml-6">
        <!-- Visible para invitados -->
        @guest
            <nav class="flex space-x-4">
                <a href="/login" class="{{ $currentRoute === 'login' ? 'text-indigo-600 font-semibold' : 'text-gray-600 hover:text-indigo-600' }}">
                    Log In
                </a>
                <a href="/register" class="{{ $currentRoute === 'register' ? 'text-indigo-600 font-semibold' : 'text-gray-600 hover:text-indigo-600' }}">
                    Registro
                </a>
            </nav>
        @endguest

        <!-- Visible para usuarios autenticados -->
        @auth
            <form method="POST" action="/logout">
                @csrf
                <button class-name="text-gray-300 hover:text-white bg-transparent hover:bg-transparent">Cerrar sesión</button>
            </form>
        @endauth
        </div>
  </div>
```

#### 5. Personalización de la página de inicio

Vamos a personalizar la página de inicio mostrando un mensaje de bienvenida si el usuario está logueado o un mensaje genérico si no lo está. Para ello vamos a utilizar el helper **auth()** para acceder a los datos del usuario logueado.

En primer lugar, creamos la ruta en `web.php`:

```php
<?php
Route::get('/', function () { return view('sections.home'); });
```

Y ahora la vista en `/resources/views/sections/home.blade.php`:

```html
@extends('layout.app')
@section('content')
<h1>Home</h1>

@auth
    <h1>Hola {{ auth()->user()->name }}!</h1>
@endauth

@guest
    <h1>Hola invitado/a!</h1>
@endguest

@endsection
```

### Resumen

- Mediante `auth()` se acceden a los datos del usuario. Ej: `auth()->user()->name;`
- Con `Auth::login($usuario)` se inicia sesión con una instancia de un usuario.
- Con `Auth::attempt(['email', 'password'])` se intenta iniciar sesión con email y password devolviendo - un boolean si ha tenido o no éxito.
- Con `Auth::logout()` se cierra la sesión.
- Se puede utilizar indistintamente el helper global `request()` o `$request` para trabajar con las peticiones. Si se hace con el segundo, hay que pasarlo explícitamente como parámetro `Request $request` en la función del controlador que se trate. Mejor práctica esta última porque mejora la legibilidad, el testing y mantenimiento.
- Con las directivas `@auth` y `@guest` en plantillas se puede mostrar contenido visible para usuarios autenticados o para invitados respectivamente.
- Utilizar `request()->session()->invalidate()` al cerrar la sesión o `request()->session()->regenerate()` en el login, son buenas prácticas que mejoran la seguridad.

## 10.2 Autorización

### Introducción

Laravel 11 ofrece un sistema de autorización flexible y potente basado en policies y gates, permitiendo restringir el acceso a diferentes partes de la aplicación según los permisos del usuario.

### Gates

Los Gates son funciones de cierre (closures) que determinan si un usuario está autorizado para realizar una acción específica.

#### Definir un Gate

Los Gates se definen en `app/Providers/AuthServiceProvider.php` dentro del método `boot()`:

```php
<?php
use Illuminate\Support\Facades\Gate;
use App\Models\User;

class AuthServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Gate::define('ver-admin', function (User $user) {
            return $user->role === 'admin';
        });
    }
}
```

#### Usar un Gate

Para verificar el permiso en un controlado:

```php
<?php
if (Gate::allows('ver-admin')) {
    // El usuario tiene permiso
}

if (Gate::denies('ver-admin')) {
    // El usuario no tiene permiso
}
```

En una vista Blade:

```html
@can('ver-admin')
    <p>Eres administrador.</p>
@endcan
```

### Policies

Las Policies son clases específicas para manejar la autorización de modelos.

#### Crear una Policy

Ejecutar el comando:

```bash
php artisan make:policy PostPolicy
```

Esto crea `app/Policies/PostPolicy.php`, donde se definen los métodos de autorización:

```php
<?php
use App\Models\User;
use App\Models\Post;

class PostPolicy
{
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

#### Registrar una Policy

En `AuthServiceProvider.php`:

```php
<?php
use App\Models\Post;
use App\Policies\PostPolicy;

protected $policies = [
    Post::class => PostPolicy::class,
];  
```

#### Usar una Policy

En un controlador:

```php
<?php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);
    // Continuar con la lógica de actualización
}
```

En una vista Blade:

```html
@can('update', $post)
    <button>Editar</button>
@endcan    
```

### Middlewares

Los middlewares permiten restringir el acceso a rutas en función de la autenticación y permisos del usuario.

#### Middleware `auth`

Para proteger rutas y asegurarse de que solo los usuarios autenticados puedan acceder:

```php
<?php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');
```

#### Middleware `can`

Para aplicar autorización específica en rutas mediante gates o policies definidas:

```php
<?php
Route::get('/admin', [AdminController::class, 'view'])->middleware('can:ver-admin');
Route::get('/post/{post}/edit', [PostController::class, 'edit'])->middleware('can:update, post');
```

#### Middleware personalizado

Se puede crear un middleware propio con Artisan:

```bash
php artisan make:middleware CheckRole
```

Esto genera `app/Http/Middleware/CheckRole.php`, donde se puede definir la lógica de autorización. Los middlewares tienen un único método llamado `handle`, que recibe una petición y una función para llamar como continuación si pasa la verificación que especifiquemos. Y si la verificación falla, podemos devolver una respuesta o redirigir a otra ruta.



```php
<?php
use Closure;
use Illuminate\Http\Request;

class CheckRole
{
    public function handle(Request $request, Closure $next, $role)
    {
        if($request->user()->role !== $role) {
            abort(403);
        }
        return $next($request);
    }
}
```
NO ES NECESARIO
Se registra en `app/Http/Kernel`.php:

```php
<?php
protected $routeMiddleware = [
    'checkRole' => \App\Http\Middleware\CheckRole::class,
];
```

Y se usa en rutas:

```php
<?php
Route::get('/admin', [AdminController::class, 'view'])->middleware('checkRole:admin');
```







### Restringir una ruta

Si nos fijamos, en el nuevo controlador que se ha creado `HomeController` podemos ver unas líneas al principio del archivo que son las que determinan si la ruta está restringida a usuarios registrados y logueados.

```php
<?php
public function __construct()
{
    $this->middleware('auth');
}
```

Mediante el uso del `middleware` llamado `auth` establecemos que todas las rutas que hagan uso de este controlador deban pasar por el login para mostrar el contenido.

Por lo tanto, en nuestros proyectos es recomendable utilizar diferentes controladores para diferentes vistas; las que estén reestringidas por el login y las que no.













