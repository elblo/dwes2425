# Uso avanzado de Frameworks

## Plantillas con Blade

## Integración de CSS y JS

## Autenticación y autorización

https://igomis.github.io/apunts/docs/4.8.Laravel.html

!!! tip Proyecto
    1. Pregunta
    2. Respuesta
    3. Relacion P-R
    4. Autenticación
    5. Relacion con usuario en modelo
    6. Relacionar con migraciones
    7. Relacionar en vistas
    8. Proteger edit, update por Auth:id
    9. página profile ... listado de preguntas y respuestas del usuario

    Utilizar la fución diffForHumas para ellapsed time

Antes se hacía así....

Run scaffolding

``` console
php artisan make:auth
```

Crea el `HomeController`, que utiliza el middleware `auth`.

### Middleware

Componente que se situa entre el enrutador y el controlador.

Explicar

Ejecutar las migraciones

Explorar los ficheros generados

Auth:routes() ???


## Laravel Breeze

Laravel Breeze es un *starter kit* que se compone de un conjunto de rutas, controladores y vistas necesarias para regitrar y autenticar usuarios en cualquier aplicación.

Las vistas están creadas con Laravel y los estilos con Tailwind CSS.

Primero hemos de añadir la dependencia mediante Composer:

``` console
composer require laravel/breeze --dev
```

Tras ello, ejecutaremos el comando `breeze:install` para generar todo el contenido necesario (rutas, vistas, controladores, recursos, ...), compilaremos todos los *assets* CSS y generaremos las migraciones:

``` console
php artisan breeze:install

npm install
npm run dev
php artisan migrate
```


!!! tip "Mailtrap"
    Para poder probar el envío de correo
    mailtrap.io, servidor de correo para pruebas para equipos de desarrollo (realmente no está envíando los correos)

## i18n










## Autenticación

Para la utenticación de usuarios necesitamos instalar unas cuantas dependencias ya preparadas para ello.

No es necesarios crear un proyecto nuevo pero nosotros vamos a hacerlo para tener uno con autenticación y otro no, el que ya hicimos al principio.

Primero de todo, vamos a crear un nuevo proyecto en Laravel que se llame `notas_auth` y nos metemos dentro de la carpeta del mismo cuando el script haya terminado.

Dentro de la carpeta `notas_auth` lanzamos los siguientes comandos.

```console
composer require laravel/ui
php artisan ui vue --auth
```

Para terminar, lanzaremos el comando `migrate` que ya conocemos... <span class="alert">**SI ESTÁS WINDOWS**</span> fuera de la imagen de Docker (utilizando xampp o parecidos) debes crear una nueva base de datos y posteriormente modificiar el archivo `.env` poniendo el nombre de esa base de datos que acabas de crear.

```console
php artisan migrate
```

Si todo ha salido bien, podrás ver en la carpeta `resources/views` una carpeta que se llama **auth** y un controlador nuevo que llama `HomeController`

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

### Datos del usuario

Siempre que queramos acceder a cualquier dato del usuario logueado, utilizaremos el método `auth()` para sacar por pantalla la información o para utilizar lógica a la hora de guardar datos en la base de datos en función de un usuario, un email o el campo que sea.

Imaginemos que tenemos una ruta donde accedemos a dicha información

```php
<?php

public function notas() {
  return auth()->user();
  
  // return auth()->user() -> name;
  // return auth()->user() -> email;
  // ...
}
```

Si visitamos esta ruta con nuestro login y password, nos aparecerá por pantalla toda la información de nuestro `user` a excepción de la contraseña y, aunque así fuera porque se lo forzamos, ésta aparecerá encriptada.
