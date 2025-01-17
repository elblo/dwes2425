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

# Gestión de datos en Laravel

> Duración estimada: 20 sesiones

## 8.1 Introducción

Laravel es un framework PHP moderno que simplifica el desarrollo de aplicaciones web, incluyendo la gestión de bases de datos. La integración con Eloquent, su ORM (Object Relational Mapping), permite trabajar con bases de datos de forma intuitiva y eficiente.

## 8.2 Configuración inicial

Laravel soporta varios motores de bases de datos como MySQL, PostgreSQL, SQLite y SQL Server. La configuración principal se realiza en el archivo `.env`. 

**Ejemplo de configuración en .env:**

```console
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nombre_base_datos
DB_USERNAME=usuario
DB_PASSWORD=password
```

Nota: El servidor MySQL debe estar funcionando con la base de datos <span class="alert">***ya creada***</span>

**Comprobar conexión:**

El siguiente comando confirma si Laravel puede conectarse a la base de datos configurada.

`php artisan migrate:status`

**Posibles problemas:**

- La extensión del driver de la base de datos (como *pdo_mysql* o *pdo_pgsql*) debe estar habilitada en el `php.ini`.
- Utilizar `php artisan config:clear` para borrar la caché de configuraciones si los cambios del `.env` no se reflejan.

**Lanzar migraciones:**

Si la conexión va bien, lanzamos las migraciones:

```console
php artisan migrate
```

Si todo ha salido bien obtendremos el siguiente resultado donde podremos observar que todas las migraciones se han insertado correctamente en la base de datos.

<div class="center img-large">
    <img src="imagenes/07/migraciones.png">
</div>

Si nos vamos al cliente que utilicemos para manejar la base de datos (phpMyAdmin por ejemplo) veremos que en nuestra base de datos se han creado todas las tablas de la migración que hemos ejecutado y **además** una tabla que se llama <span class="success">***migrations***</span>.

La tabla `migrations` es simplemente un registro de todas las migraciones llevadas a cabo.

## 8.3 Migraciones

### Introducción

Las migraciones son un sistema de control de versiones para bases de datos que permite trabajar de forma colaborativa, manteniendo un histórico de los cambios realizados en el esquema. Con las migraciones se puede: 

- Crear, modificar y borrar tablas. 
- Gestionar el esquema de forma programática utilizando **Artisan** y el **Schema Builder**. 
- Revertir cambios mediante **rollback** o volver a aplicar todos los cambios con **refresh**.

### Estructura de las migraciones

Las migraciones de un proyecto Laravel se guardan en el directorio *database/migrations* en archivos `.php` y siguen una estructura predefinida con dos métodos principales: 

- **up**: Define las operaciones que deben aplicarse en la base de datos (crear tablas, añadir columnas, etc.). 
- **down**: Define las operaciones inversas para revertir (rollback) los cambios aplicados por up.

Ejemplo:

```php
<?php
public function up()
{
    Schema::create('usuarios', function (Blueprint $tabla) {
        $tabla->id();
        $tabla->string('nombre');
        $tabla->string('email')->unique();
        $tabla->timestamps();
    });
}

public function down()
{
    Schema::dropIfExists('usuarios');
}
```

Por defecto, Laravel añade un campo autonumérico *id* y si se llama al método `timestamps()`, dos columnas *created_at* y *updated_at* que se actualizan automáticamente para saber cuándo se creó y actualizó un registro.

### Crear una migración

Mediante el comando `make:migration` de Artisan generamos una migración, un archivo con las instrucciones (Schema Builder) para construir o cambiar las tablas de la base de datos. En el nombre de dicho archivo se incluye un timestamp para asegurar el orden cronológico.

**Ejemplos:**

```bash
# Migración en blanco
php artisan make:migration nombre_migración 

# Migración para crear una tabla
php artisan make:migration create_table_usuarios --create=usuarios  

# Migración para modificar una tabla
php artisan make:migration add_fields_to_usuarios --table=usuarios 
```

Laravel puede inferir acciones del nombre de la migración gracias a la clase **TableGuesser**. Por ejemplo, si el nombre contiene *create* o *to*, Artisan deducirá si es para crear o modificar tablas.

### Ejecutar migraciones

- `php artisan migrate`: Ejecuta las migraciones pendientes.
- `php artisan migrate:status`: Muestra el estado de las migraciones.
- `php artisan migrate:fresh`: Borra todas las tablas de la BDD (sin ejecutar rollback) y ejecuta todas las migraciones.
- `php artisan migrate:refresh`: Hace un rollback de todas las migraciones y las vuelve a ejecutar. Para rellenar la BDD con datos de prueba, usar el flag --seed.
- `php artisan migrate:reset`: Hace un rollback de todas las migraciones.
- `php artisan migrate:rollback`: Revierte la la última migración.

### Schema Builder

La clase **Schema** es el kernel para definir y modificar el esquema de las bases de datos. Incluye constructores para crear, modificar y eliminar tablas y columnas. Y es lo que utilizaremos dentro de los archivos de migraciones.

#### Crear y eliminar tablas

```php
<?php
Schema::create('usuarios', function (Blueprint $table) {
    $table->id();
    $table->string('nombre', 32);
    $table->timestamps();
});

Schema::dropIfExists('usuarios');
```

#### Añadir y eliminar columnas

```php
<?php
Schema::table('usuarios', function (Blueprint $table) {
    $table->string('telefono')->after('nombre')->nullable();
});

Schema::table('usuarios', function (Blueprint $table) {
    $table->dropColumn('telefono');
});
```

#### Tipos de columnas

Laravel ofrece una amplia variedad de tipos de columnas que puedes consultar en la [documentación oficial](https://laravel.com/docs/11.x/migrations#available-column-types).

#### Índices

```php
<?php
$table->primary('id'); // Campo id como clave primaria
$table->primary(['nombre', 'apellidos']); // Clave primaria compuesta
$table->unique('email'); // Campo email único
$table->index('localidad'); // Campo localidad como índice
```

#### Claves foráneas

```php
<?php
Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('usuarios');
});
```

Laravel proporciona mediante el método `foreignId` una forma más concisa de hacer lo anterior, creando automáticamente el *unsignedBigInteger* y determinando la tabla a la que hace referencia (user) por el nombre del campo.

```php
<?php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

Si la tabla a la que hace referencia no sigue las convenciones de nombres de Laravel, se puede indicar a mano la referencia.

```php
<?php
$table->foreignId('user_id')->constrained(
    table: 'usuarios', indexName: 'id_usuario'
);
```

Y también se puede especificar si queremos que los registros de la tabla actual se actualicen o borren en cascada según lo haga el registro de la tabla principal.

```php
<?php
$table->foreignId('user_id')
      ->constrained()
      ->onUpdate('cascade')
      ->onDelete('cascade');
```

## 8.4 Query Builder

El **Query Builder** de Laravel proporciona una interfaz fluida para construir y ejecutar consultas de bases de datos. Permite trabajar con varias bases de datos de forma sencilla sin escribir código SQL.

Es ideal para crear *consultas personalizadas* en las que el rendimiento es una prioridad y *consultas complejas* que no se pueden expresar fácilmente con Eloquent.

##### Ejemplos de consultas

```php
<?php
// Obtener todos los registros de users
$users = DB::table('users')->get(); 

// Filtrar registros
$users = DB::table('users')->where('type', 'customer')->get();

// Seleccionar columnas
$users = DB::table('users')->select('name', 'email')->get();

// Ordenar resultados
$users = DB::table('users')->orderBy('name', 'asc')->get();

// Contar registros
$count = DB::table('users')->count();

// Agregados
$maxSalary = DB::table('employees')->max('salary');

// Subconsultas
$users = DB::table('users')
    ->whereExists(function($query) {
        $query->select(DB::raw(1))
              ->from('orders')
              ->whereColumn('orders.user_id', 'users.id');
    })
    ->get();
```

##### Ejemplos de manipulación de datos

```php
<?php
// Insertar registro
DB::table('users')->insert([
    'name' => 'John Doe',
    'email' => 'john@example.com',
]);

// Actualizar registro
DB::table('users')
    ->where('id', 1)
    ->update(['name' => 'Updated Name']);

// Eliminar registro
DB::table('users')
    ->where('id', 1)
    ->delete();

// Borrar todos los registros
DB::table('users')->truncate();
```

## 8.5 Eloquent: Modelos

### Introducción

Un **ORM (Object-Relational Mapping)** es una técnica de programación que permite eliminar la disparidad entre el modelo de datos de una base de datos relacional y el modelo de objetos de una aplicación. Mientras que en una BDD pensamos en tablas y campos, en el mundo de desarrollo pensamos en objetos y propiedades.

Ventajas de un ORM:

- **Abstracción de la base de datos**: No es necesario escribir SQL, ya que el ORM se encarga de traducir las operaciones de la base de datos a objetos.
- **Nombres de campos y tablas**: No es necesario recordar los nombres de las tablas y campos, ya que el ORM se encarga de ello. Si cambiamos el nombre de un campo, solo tenemos que cambiarlo en un lugar, en el modelo.
- **Relaciones**: Las relaciones entre tablas se pueden definir en los modelos, y el ORM se encarga de gestionarlas. Atravesar relaciones es tan sencillo como acceder a una propiedad de un objeto.

**Eloquent** es el ORM de Laravel, y nos permite interactuar con la base de datos de una forma sencilla y elegante. Eloquent es una capa de abstracción de la base de datos, que nos permite interactuar con ella utilizando objetos. 

Cada tabla de la base de datos tiene un modelo asociado, que es una clase que representa a la tabla. Por tanto:

- las tablas son modelos
- los registros de la tabla son instancias de ese modelo
- los campos de una tabla son propiedades del modelo

<figure style="align: center;">
    <img src="imagenes/08/eloquent-orm.png" width="700">
    <figcaption>Relación entre tabla y modelo</figcaption>
</figure>

### Modelos

Los modelos son uno de los componentes más importantes de Laravel, son los responsables de interactuar con nuestra base de datos de una manera orientada a objetos. Representan las tablas de la base de datos como clases en la aplicación, permiten realizar operaciones para seleccionar, crear, actualizar y eliminar datos de una manera más sencilla y estructurada.

#### Crear un modelo

Los modelos se definen dentro de la carpeta *app/Models* y se pueden crear mediante Artisan:

```console
php artisan make:model Nota -m
```

!!! danger "Nombrar correctamente"
    El nombre del modelo empieza por Mayúscula y siempre se escribe en **SINGULAR***. 
    
    Si le pasamos el parámetro **-m** además creará la migración con el código para crear la tabla correspondiente en la BDD, cuyo nombre irá en minúscula y plural.

??? info "Opciones al crear el modelo"
    Podemos añadir las siguientes opciones al comando para crear otros elementos relacionados con el modelo:

    - **-c**, --controller: Crea un controlador.
    - **-m**, --migration: Crea una migración.
    - **-r**, --resource: Crea un controlador y una vista.
    - **-f**, --factory: Crea un factory.
    - **-s**, --seed: Crea un seeder.
    - **-a**, --all: Crea un controlador, una migración y una vista.
  
    Por ejemplo, si queremos crear un modelo, una migración y un controlador, ejecutamos:

    ```console
    php artisan make:model Nota -cm 
    ```

Si todo ha salido bien, veremos en nuestro directorio de migraciones `database/migrations` un nuevo archivo con un nombre similar a `2025_01_21_111237_create_notas_table.php` en el que se encuentra la tabla relacionada y que podemos abrir para seguir añadiendo campos mediante el **Schema Builder** como se ha visto anteriormente. Por ejemplo:

```php
<?php

Schema::create('notas', function (Blueprint $table) {
  $table->id();
  $table->timestamps();
  // Campos añadidos
  $table->string('nombre'); 
  $table->text('descripcion');
  $table->integer('prioridad');
});
```

Una vez tengamos listo nuestro esquema debemos lanzar `php artisan migrate` para que ejecute las migraciones pendientes introduciendo la nueva información en la base de datos.

#### Uso básico de un modelo

##### Recuperar datos

```php
<?php
// Todos los registros
$notas = Nota::all();

// Registros filtrados
$notas = Nota::where('prioridad', '>', 5)->get();

// Registro único
$nota = Nota::findOrFail($id);
```

##### Insertar datos
```php
<?php
$nota = new Nota();
$nota->titulo = "Proyecto Laravel";
$nota->descripcion = "Programar la parte de los modelos de la práctica de Fútbol Femenino.";
$nota->prioridad = 10;
$nota->save();
```

##### Actualizar datos
```php
<?php
$nota = Nota::find($id);
$nota->titulo = "Nuevo título";
$nota->save();
```

##### Eliminar datos
```php
<?php
$nota = Nota::find($id);
$nota->delete();
```

#### Propiedades comunes de los modelos Eloquent

En los modelos podemos definir varias propiedades para configurar el comportamiento de la interacción con la base de datos. A continuación se detallan las más importantes:

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    // Especifica el nombre de la tabla si no sigue la convención de nombres de Laravel
    protected $table = 'custom_table_name';

    // Define la clave primaria de la tabla
    protected $primaryKey = 'custom_id';
    // Indica si la clave primaria es autoincremental
    public $incrementing = false;
    // Especifica el tipo de la clave primaria (si no es integer)
    protected $keyType = 'string';

    // Define qué atributos pueden ser asignados masivamente (a través de métodos como create o update)
    protected $fillable = ['name', 'email', 'password'];
    // Contrario a $fillabel. Define qué atributos no pueden ser asignados masivamente
    protected $guarded = ['is_admin'];

    // Define los atributos a ocultar al serializar el modelo (a JSON o array)
    protected $hidden = ['password', 'remember_token'];
    // Contrario a $hidden, define los atributos que serán visibles al serializar
    protected $visible = ['name', 'email'];

    // Transformación automática de los atributos a un tipo específico
    protected $casts = [
        'is_admin' => 'boolean',
        'settings' => 'array',
    ];

    // Indica si la tabla tiene los campos `created_at` y `updated_at`
    public $timestamps = true;

    // Defineix la conexión a la BDD
    protected $connection = 'mysql';
}
```

### Ejemplo recuperar datos

Ya tenemos nuestra base de datos creada con las tablas migradas, ahora sólo falta rellenarlas con algunos datos de prueba a través del cliente de MySQL que más nos guste:

- PHPMyAdmin
- [MySQL Workbench](https://www.mysql.com/products/workbench/)
- [HeidiSQL](https://www.heidisql.com/download.php)

Vamos a ver cómo recuperar esos datos desde una vista y lo primero que vamos a necesitar va a ser un controlador para gestionar esas notas:

```bash
php artisan make:controller NotaController
```

Es recomendable seguir la convención de Laravel de nombres de rutas y funciones de controladores que viemos [aquí](https://elblo.github.io/dwes2425/07frameworks.html#controlador-de-recursos).

##### 1. Rutas 

Creamos la ruta en `web.php` que redirige a la función correspondiente del controlador para mostrar todas las notas o una en particular.

```php
<?php
// estamos en ▓▓▓ web.php 
Route::get('notas', [ PagesController::class, 'index' ])->name('notas.index');
Route::get('notas/{id?}', [ PagesController::class, 'show' ])->name('notas.show');
```

##### 2. Controlador

Desde la función del controlador se llama al modelo para recuperar las notas y devuelve la vista pasándoselas.

```php
<?php
// estamos en ▓▓▓ NotaController.php 

// Muestra listado de notas
public function index() {
  $notas = Nota::all();
  return view('notas.index', compact('notas'));
}

// Muestra una nota en específico
public function show($id) {
  $nota = Nota::findOrFail($id);
  return view('notas.show', compact('nota'));
}
```

##### 3. Vistas

`notas/index.blade.php`: Vista con la tabla que pinta los datos mediante las notas pasadadas como parámetro.

```html
<h1>Notas desde base de datos</h1>

@if(session('mensaje'))
    <div>{{ session('mensaje') }}</div>
@endif

<table border="1">
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Prioridad</th>
            <th>Editar</th>
            <th>Eliminar</th>
        </tr>
    </thead>
    
    @foreach ($notas as $nota)
        <tr>
            <td>{{$nota->titulo}}</td>
            <td>{{$nota->descripcion}}</td>
            <td>{{$nota->prioridad}}</td>
            <td>📝</td>
            <td>❌</td>
        </tr>
    @endforeach
</table>
<p><a href="">Nueva nota</a></p>

```

``notas/show.blade.php`: Vista con el detalle de una nota en particular. Hace uso de la plantilla que teníamos y está dentro de la subcarpeta *notas*.

```php
<?php
// estamos en ▓▓▓ notas/show.blade.php
@extends('plantilla')

@section('apartado')
  <h1>Detalle de la nota</h1>

  <p>ID: {{ $nota->id }}</p>
  <p>Nombre: {{ $nota->titulo }}</p>
  <p>Descripción: {{ $nota->descripcion }}</p>    
  <p>Prioridad: {{ $nota->prioridad }}</p>     
@endsection
```

!!! info "Ojo con los nombres"
    Hay que fijarse bien en los nombres de las columnas que tienen nuestras tablas, porque serán los nombres de los atributos de los objetos del modelo que utilizaremos.

### Modificar tablas sin perder datos

A veces cometemos errores de diseño y queremos introducir una nueva columna dentro de nuestra tabla o modificar una de esas columnas <span class="alert">***SIN PERDER LOS DATOS DE LA BASE DE DATOS***</span>.

Imaginemos que en nuestra tabla `notas` queremos agregar una columna con el nombre `autor`.

Lo primero de todo es crear una nueva migración para realizar este cambio mediante *Artisan* con el nombre `add_fields_to_` seguido del nombre de la tabla a modificar.

```console
php artisan migrate add_fields_to_nota
```

Seguidamente, abrimos el archivo de la migración que acabamos de crear y en la función `up()` ponemos el cambio que queremos realizar y en `down()` lo eliminamos para que en caso de hacer una migración rollback, se vuelva a quedar todo como estaba.

```php
<?php

public function up()
{
  Schema::table('notas', function (Blueprint $table) {
      $table->string('autor');
  });
}

public function down()
{
  Schema::table('notas', function (Blueprint $table) {
      $table->dropColumn('autor');
  });
}
```

## 8.6 Formularios

Ya sabemos cómo recuperar datos de una base de datos. Ahora toca ver cómo insertarlos, actualizaros y eliminarlos con Laravel y sin escribir ni una sola línea de SQL.

Es recomendable seguir la convención de Laravel de nombres de rutas y funciones de controladores que viemos [aquí](https://elblo.github.io/dwes2425/07frameworks.html#controlador-de-recursos).

### Insertar datos

Para insertar datos vamos a necesitar 2 rutas, 2 funciones en el controlador y 1 vista con el formulario:

- Con la primera ruta `notas/create` llamaremos a la función *create* del controlador que abrirá el formulario para crear una nueva nota.
- El formulario enviará los datos a la segunda ruta `notas` mediante **POST**, la cual llamará a la función *store* del controlador para crear la nota mediante el método *save()*.

##### 1. Rutas

Creamos las rutas GET y POST con sus alias correspondientes en nuestro archivo de rutas `web.php`. Situar la ruta `notas/create` antes de la ruta `notas/{id?}` porque si no entraría siempre en esta última.

```php
<?php
// estamos en ▓▓▓ web.php
Route::get('notas/create', [ NotaController::class, 'create' ])->name('notas.create');
Route::post('notas', [ NotaController::class, 'store' ])->name('notas.store');
```

##### 2. Controlador

En el controlador creamos los 2 métodos:
- `create` para abrir el formulario.
- `store` para crear la nueva nota con los datos que le llegan del formulario mediante *Request* y almacenarla medidante *save()* y volvemos a la página del formulario con el método *back()* añadiendo un mensaje con *with()*.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Muestra el formulario para crear una nueva nota
public function create() {
    return view('notas.create');
}

// Crea una nota con la info del formuluario
public function store(Request $request) {
    $notaNueva = new Nota();
    $notaNueva->titulo = $request->titulo;
    $notaNueva->descripcion = $request->descripcion;
    $notaNueva->prioridad = $request->prioridad;
    $notaNueva -> save();

    // Volver al formulario para seguir insertando
    return back()->with('mensaje', 'Nota insertada');
}
```

##### 3. Vista

`notas/create.blade.php`: Vista con el formulario para crear una nueva nota. En el *action* se indica la ruta a la que enviar los datos por POST.

- En el *action* se indica la ruta a la que enviar los datos por POST.
- El atributo *name* de los inputs tiene que ser igual al del campo correspondiente de la tabla.
- Se usa la cláusula de seguridad `@csrf` para evitar ataques desde otros sitios. [Más info](https://www.ionos.es/digitalguide/servidores/seguridad/cross-site-request-forgery/) sobre este ataque.
- Con *session('mensaje')* mostramos el mensaje que viene del controlador.

```html
<h2>Crear nueva nota</h2>
@if (session('mensaje'))
    <div class="mensaje-nota-creada">{{ session('mensaje') }}</div>
@endif

<form action="{{ route('notas.store') }}" method="POST">
    @csrf {{-- Cláusula para obtener un token de formulario al enviarlo --}}
    <div>
        <input type="text" name="titulo" placeholder="Título de la nota" class="form-control mb-2" autofocus />
        <input type="text" name="descripcion" placeholder="Descripción de la nota" class="form-control mb-2" />
        <input type="number" name="prioridad" placeholder="5" class="form-control mb-2" />

        <button class="btn btn-primary btn-block" type="submit">Crear nueva nota</button>
    </div>
</form>

<div><a href="{{ route('notas.index') }}" class="btn btn-success btn-block mt-2">Volver</a></div>
```

##### 4. Incluir enlace para insertar

En la vista `notas/index.blade.php` añadimos un enlace o botón que abra el formulario para crear una nueva nota.

```html
<p><a href="{{ route('notas.create') }}">Nueva nota</a></p>
```

### Actualizar datos

Para actualizar, al igual que para insertar datos, vamos a necesitar 2 rutas, 2 funciones en el controlador y 1 vista con el formulario:

- Con la primera ruta `notas/{id}/edit` llamaremos a la función *edit* del controlador que abrirá el formulario para modificar la nota.
- El formulario enviará los datos a la segunda ruta `notas/{id}` mediante **PUT**, la cual llamará a la función *update* del controlador para actualizar la nota mediante el método *save()*.

##### 1. Rutas

Creamos las rutas GET y PUT con sus alias correspondientes en nuestro archivo de rutas `web.php`.

```php
<?php
// estamos en ▓▓▓ web.php
Route::get('notas/{id}/edit', [ NotaController::class, 'edit' ])->name('notas.edit');
Route::put('notas/{id}', [ NotaController::class, 'update' ])->name('notas.update');
```

##### 2. Controlador

En el controlador creamos los 2 métodos:
- `edit` para abrir el formulario.
- `update` para actualizar la nota con los datos que le llegan del formulario mediante *Request* y almacenarla medidante *save()* y volvemos a la página del formulario con el método *back()* añadiendo un mensaje con *with()*.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Muestra el formulario para editar una nota
public function edit($id) {
    $nota = Nota::findOrFail($id);
    return view('notas.edit', compact('nota'));
}

// Almacena la info recibida del formulario de edición
public function update(Request $request, $id) {
    $notaUpdate = Nota::findOrFail($id);
    $notaUpdate->titulo = $request->titulo;
    $notaUpdate->descripcion = $request->descripcion;
    $notaUpdate->prioridad = $request->prioridad;
    $notaUpdate->save();

    // Volver al listado de notas
    //return redirect('/notas')->with('mensaje','Nota actualizada');
    return redirect()->route('notas.index')->with('mensaje','Nota actualizada');
}
```

##### 3. Vista

`notas/edit.blade.php`: Vista con el formulario para actualizar la nota. En el *action* se indica la ruta a la que enviar los datos por POST.

- En el *action* se indica la ruta a la que enviar los datos junto al *id* de la nota.
- Mediante `@method('PUT')` indicamos que se haga la petición a la url del formulario mediante el método **PUT**, que es como la recogemos en las rutas.
- Se usa la cláusula de seguridad `@csrf` para evitar ataques desde otros sitios. [Más info](https://www.ionos.es/digitalguide/servidores/seguridad/cross-site-request-forgery/) sobre este ataque.
- El atributo *name* de los inputs tiene que ser igual al del campo correspondiente de la tabla.
- Con *session('mensaje')* mostramos el mensaje que viene del controlador.

```html
<h2>Editando la nota {{ $nota -> id }}</h2>

<form action="{{ route('notas.update', $nota->id) }}" method="POST">
    @method('PUT') {{-- Necesitamos cambiar al método PUT para editar --}}
    @csrf {{-- Cláusula para obtener un token de formulario al enviarlo --}}

    @error('nombre')
        <div class="alert alert-danger">El nombre es obligatorio</div>
    @enderror

    @error('descripcion')
        <div class="alert alert-danger">La descripción es obligatoria</div>
    @enderror

    @error('prioridad')
        <div class="alert alert-danger">La descripción es obligatoria</div>
    @enderror

  <input
      type="text"
      name="titulo"
      class="form-control mb-2"
      value="{{ $nota->titulo }}"
      placeholder="Título de la nota"
      autofocus
  />
  <input
      type="text"
      name="descripcion"
      placeholder="Descripción de la nota"
      class="form-control mb-2"
      value="{{ $nota->descripcion }}"
  />
  <input
      type="number"
      name="prioridad"
      placeholder="5"
      class="form-control mb-2"
      value="{{ $nota->prioridad }}"
  />

  <button class="btn btn-primary btn-block" type="submit">Guardar cambios</button>
</form>

<div><a href="{{ route('notas.index') }}" class="btn btn-success btn-block mt-2">Volver</a></div>
```

##### 4. Incluir enlace para editar

En la vista `notas/index.blade.php` añadimos a cada nota que se muestra en la tabla un enlace para poder editarla.

```html
<td><a href="{{ route('notas.edit', $nota->id) }}">📝</a></td>
```

### Eliminar datos

Para insertar datos vamos a necesitar 1 ruta y 1 función en el controlador:

- Con la ruta `notas/{id}` que llega mediante **DELETE** llamaremos a la función *destroy* del controlador.

##### 1. Ruta

Creamos las rutas DELETE con su alia correspondiente en nuestro archivo de rutas `web.php`.

```php
<?php
// estamos en ▓▓▓ web.php
Route::delete('notas/{id}', [ NotaController::class, 'destroy' ])->name('notas.destroy');
```

##### 2. Controlador

En el controlador creamos el método:
- `destroy` que mediante el método *delete()* elimina la nota y redirige al listado de notas añadiendo un mensaje con *with()*.

```php
<?php
// estamos en ▓▓▓ NotaController.php

public function destroy($id) {
    $notaEliminar = Nota::findOrFail($id);
    $notaEliminar->delete();
  
    // Volver al listado de notas
    return back()->with('mensaje','Nota eliminada');
}
```

##### 3. Vista

No es necesaria ninguna vista específica 🥳

##### 4. Incluir enlace para eliminar

En la vista `notas/index.blade.php` añadimos un botón para cada nota que lance la ruta con el id de la nota a eliminar.

```html
<td>
    <form action="{{ route('notas.destroy', $nota->id) }}" method="POST">
        @method('DELETE')
        @csrf
        <button type="submit">❌</button>
    </form>
</td>
```

!!! info "Enhorabuena!"
    Si todo ha salido bien, habrás creado un sitio en Laravel y Eloquent capaz de hacer un ***CRUD*** con datos reales en una base de datos.

## 8.7 Validación

Laravel nos proporciona herramientas para poder validar los datos que el usuario introduce en los campos del formulario.

Además de poder hacerlo con la etiqueta `required` de HTML5, debemos validar los datos a través del Framework.

Para ello, necesitamos modificar varios elementos:

  - En primer lugar, nuestro archivo `controller`
  - En segundo lugar, nuestra `plantilla` que carga el formulario

Empecemos con el controlador. A través del método `validate()` le decimos a Eloquent qué campos son requeridos para poder enviar el formulario. Utilizaremos para ello un array asociativo con el nombre del input y la palabra reservada `required`

```php
<?php

// estamos en ▓▓▓ PagesController.php

$request -> validate([
  'nombre' => 'required',
  'descripcion' => 'required'
]);
```

Seguidamente nos moveremos a la plantilla donde esté el formulario y a través de la directiva `@error` crearemos un bloque html con nuestro mensaje de error por cada uno de los inputs requeridos.

```php
<?php

// estamos en ▓▓▓ notas.blade.php

@error('nombre')
    <div class="alert alert-danger">
      No olvides rellenar el nombre
    </div>
@enderror

```

Pero ¿qué pasa cuando ha habido un error y nos muestra el mensaje que hemos escrito? Si te fijas, los campos que habías rellenado perderán la información, pero con Laravel podemos persistirlos sin hacer que el usuario vuelva a introducirlos.

Para poder persistir los datos una vez enviados pero con algún error de campo requerido, utilizaremos la directiva `old()` como value del input dentro de nuestro formulario y le pasaremos el nombre del input declarado en la etiqueta `name`.

```php
<?php

// estamos en ▓▓▓ notas.blade.php

<input
  type="text"
  name="nombre"
  value="{{ old('nombre') }}"
  class="form-control mb-2"
  placeholder="Nombre de la nota"
  autofocus
>
```

## 8.8 Paginación

Para añadir paginación a nuestros resultados, Eloquent tiene un método que se llama `paginate()` donde le pasamos un número entero como parámetro para indicarle el número de resultados que queremos por página.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Muestra listado de notas
public function index() {
    $notas = Nota::paginate(3); // 3 notas por página
    return view('notas.index', compact('notas'));
}
```

Y en la vista `notas/index.blade.php`, para que muestre los enlaces para pasar de página incluimos:

```html
<p>{{ $notas->links() }}</p>
```

La librería de paginación que utiliza Laravel está situada en la carpeta `vendor/laravel/framework/src/illuminate/Pagination`. Abriendo su archivo `resources/views/tailwind.blade.php` se puede ver la estructura HTML del sistema de paginación. 

Aunqeu no es recomendable tocar la carpeta *vendor*, por practicar podríais modificar ese archivo (guardando antes una copia del mismo).

## 8.9 Eloquent: Relaciones

A través de Eloquent vamos a poder gestionar las relaciones entre nuestras tablas de la base de datos de una manera muy sencilla y sin sentencias SQL.

### Uno a uno (1 a 1)

Para crear este tipo de relaciones en Eloquent y Laravel, debemos tener creadas las tablas que vayamos a relacionar y establecer la relación entre ellas a través del método `hasOne`.

Supongamos que tenemos una tablas `usuario` que está relacionada con la tabla `telefono`.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Usuario extends Model
{
  /**
   * Obtener el Teléfono asocioado con el Usuario
   */
  public function telefono()
  {
      return $this -> hasOne(Telefono::class);
  }
}

```

Una vez hecho ésto, para poder recuperar el dato relacionado, debemos utilizar las propiedades dinámicas de Eloquent. Con estas propiedades dinámicas podremos obtener dicho dato.

```php
<?php

$telefono = Usuario::find(1)->telefono;
```

En este caso, Eloquent asume que en `Usuario` existe la clave ajena `usuario_id` pero ¿qué pasa si tenemos otro nombre? pues se lo pasamos como parámetro.

```php
<?php

return $this->hasOne(Telefono::class, 'clave_ajena');
```

### Uno a Uno ***INVERSA***

Ahora que podemos acceder al modelo teléfono desde el modelo usuario, vamos a ver cómo hacerlo de manera inversa, es decir, cómo acceder desde el módelo `usuario` desdel el modelo `telefono` gracias al método `belongsTo()`.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Telefono extends Model
{
  public function usuario()
  {
      return $this -> belongsTo(Usuario::class);
  }
}
```

Al llamar el método de `usuario`, Eloquent intentará encontrar un modelo de usuario que tenga un `id` que coincida con la columna de `usuario_id` en el modelo de `telefono`.

Eloquent determina el nombre de la clave externa examinando el nombre del método de relación y agregando el sufijo `_id` al nombre del método. Entonces, asume que el modelo `Telefono` tiene una columna `usuario_id`. Sin embargo, si no se llama de esa manera, puedes pasarle como argumento el nombre de la clave.

```php
<?php

public function usuario()
{
    return $this -> belongsTo(Usuario::class, 'clave_ajena');
}
```

### Uno a Muchos (1 a MM)

En este caso, las relaciones de 1 a muchos podemos decir que en una entrada de un blog, o en un post de Facebook, hay muchos comentarios relacionados a esa misma publicación.

Para empezar, ya sabemos que debemos crear el modelo y en este caso usaremos el método `hasMany()` para obtener los datos relacionados con ese post o entrada en el blog

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
  
  public function comentarios()
  {
      return $this -> hasMany(Comentario::class);
  }
}
```

Cuidado con las claves ajenas, que aquí pasa lo mismo... Eloquent establece por defecto el sufijo `_id` por lo tanto, en este ejemplo buscaría por `post_id`. Si no queremos éso o nuestra clave ajena tiene otro nombre, se lo pasamos por parámetro en el método `hasMany` como hacíamos más arriba.

Ahora, al haber más de un dato, necesitamos iterar, por tanto debemos crear un bucle para poder sacar cada dato.

```php
<?php
use App\Models\Post;

$comentarios = Post::find(1) -> comentarios;

foreach ($comentarios as $comentario) {
    // Lo que sea que hagamos con esos datos
}
```

Además, como todas las relaciones son sentencias SQL, podemos anidar varios filtros en función de lo que queramos sacar.

```php
$comentario = Post::find(1) -> comentarios()
    ->where('titulo', 'lo que sea')
    ->first();
```

### Uno a Muchos ***INVERSA***

Ahora que podemos acceder a todos los comentarios de una publicación, definamos una relación para permitir que un comentario acceda a su publicación principal.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Comentario extends Model
{

  public function post()
  {
      return $this -> belongsTo(Post::class);
  }
}
```

Y ahora, a través de la propiedad de relación dinámica...

```php
<?php

use App\Models\Comentario;

$comentario = Comentario::find(1);

return $comentario -> post -> titulo;
```

Pasaría lo mismo con el nombre de la clave ajena, si no se llama de la misma manera que Eloquent establece con el sufijo `_id` podemos pasarle como parámetro el nombre de la clave donde debe buscar.

### Muchos a Muchos (MM a MM)

Este tipo de relaciones son las más complicadas ya que, en un Blog del estilo Wordpress por ejemplo, un usuario puede tener muchos roles (lector, autor, administrador) pero un rol pueden tenerlo varios usuarios, es decir, puede haber muchos usuarios administradores, otros lectores y demás.

Para realizar este tipo de relaciones necesitaríamos 3 tablas diferentes.

  - usuarios [ id, nombre]
  - roles [id, nombre]
  - rol_usuario [usuario_id, rol_id] (Tabla Pivote)

Lo primero de todo, vamos a crear las tablas con sus modelos <span class="alert">***a excepción de la tabla pivote rol_usuario***</span> que <span class="warning">***sólo crearemos la tabla, sin su modelo***</span>

```console
php artisan make:migration create_rol_usuario_table --create=rol_usuario
```

Y la estructura de dicha seria de la siguiente manera...

```php
<?php

public function up()
{
    Schema::create('rol_usuario', function (Blueprint $table) {
        $table->bigIncrements('id');
        $table->unsignedInteger('usuario_id');
        $table->unsignedInteger('rol_id');
        $table->timestamps();
    });
}
```

Ahora que ya tenemos todo listo, las relaciones de Muchos a Muchos vienen definidas por un método que devuelve el resultado de usar el método `belongsToMany()`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Usuario extends Model
{
    public function roles()
    {
        return $this -> belongsToMany(Rol::class);
    }
}
```

Una vez que tengamos las relaciones definidas, accederemos a ellas mediante las propiedades dinámicas de `rol`

```php
<?php

use App\Models\Usuario;

$usuario = Usuario::find(1);

foreach ($usuario -> roles as $rol) {
    // nuestro código
}
```

Acordaros que podemos encadenar comandos sql a través de los métodos de Eloquent

```php
<?php

$roles = Usuario::find(1) -> roles() -> orderBy('nombre') -> get();
```

### Muchos a Muchos ***INVERSA***

Para definir el "inverso" de una relación de muchos a muchos, debemos establecer un método en el modelo relacionado que también devuelva el resultado del método `belongsToMany `. Según el ejemplo que estamos siguiendo...

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Rol extends Model
{
  public function usuarios()
  {
      return $this -> belongsToMany(Usuario::class);
  }
}
```

Vamos a hacer un ejemplo con una APP que gestiones alumnos y asignaturas, de tal manera que MUCHOS ALUMNOS pueden cursar MUCHAS ASIGNATURAS

¿Qué necesitamos para este ejemplo?

  - 3 migraciones para crear las tablas
    - `Alumnos` /// `Materias` /// `AlumnoMateria`

  - Modificar los archivos de las migraciones `create_alumnos_table` y `create_materias_table`.
  - Crear la base de datos `muchos_a_muchos`
  - Ejecutar las Migraciones
  - 2 **modelos** para `Alumnos` /// `Materias`
  - Método dentro de `Alumno` para crear la relación Alumno -> Materia
  - Crear el controlador para la vista
  - Crear la ruta de nuestra vista
  - Rellenar la base de datos
  - Crear la vista con los datos
  
<span class="success">**3 MIGRACIONES**</span>
```console
php artisan make:migration create_alumnos_table
php artisan make:migration create_materias_table
php artisan make:migration create_alumno_materia_table
```

<span class="success">**MODIFICANDO LAS MIGRACIONES**</span>

=== "create_alumnos_table.php"

    ``` php
    <?php

    public function up()
    {
      Schema::create('alumnos', function (Blueprint $table) {
        $table->id();
        $table->string('nombre');
        $table->timestamps();
      });
    }
    ```
=== "create_materias_table.php"

    ``` php
    <?php

    public function up()
    {
      Schema::create('materias', function (Blueprint $table) {
        $table->id();
        $table->string('nombre');
        $table->timestamps();
      });
    }
    ```

=== "create_alumno_materia_table.php"

    ``` php
    <?php

    public function up()
    {
      Schema::create('alumno_materia', function (Blueprint $table) {
        $table->id();

        $table->foreignId('alumno_id')
          ->nullable()
          ->constrained('alumnos')
          ->cascadeOnUpdate()
          ->nullOnDelete();

        $table->foreignId('materia_id')
          ->nullable()
          ->constrained('materias')
          ->cascadeOnUpdate()
          ->nullOnDelete();

        $table->timestamps();
      });
    }
    ```

<span class="success">**CREAMOS LA BASE DE DATOS**</span>

Para este ejemplo, vamos a crear una base de datos que se llame `muchos_a_muchos` desde la consola de MySQL o MariaDB.

``` console
CREATE TABLE `muchos_a_muchos`
```

<span class="success">**EJECUTANDO LAS MIGRACIONES**</span>

Ya tenemos las migraciones creadas y la base de datos lista para insertar el contenido de las migraciones que hemos escrito más arriba, lo que nos queda es `ejecutar las migraciones` para volcar toda la estructura en nuestra nueva base de datos.

``` console
php artisan migrate
```

<span class="success">**2 MODELOS PARA ALUMNOS Y MATERIAS**</span>

``` console
php artisan make:model Alumno
php artisan make:model Materia
```

<span class="success">**MÉTODOS PARA CREAR LAS RELACIONES ALUMNO <-> MATERIA**</span>

=== "Alumno.php"

    ``` php
    <?php

    public function materias() {
      return $this -> belongsToMany(Materia::class, 'alumno_materia');
    }
    ```

=== "Materia.php"

    ``` php
    <?php

    public function alumnos() {
      return $this -> belongsToMany(Alumno::class, 'alumno_materia');
    }
    ```

<span class="success">**CREANDO EL CONTROLADOR DE LA VISTA**</span>

Necesitamos un controlador para redireccionar las rutas a las vistas que nosotros queramos, para ello crearemos el controlador `RelacionController`

```console
php artisan make:controller RelacionController
```


<span class="success">**CREANDO RUTAS**</span>

Ahora que ya tenemos nuestro controlador, vamos a crear una única vista para mostrar el ejemplo de la relación MUCHOS a MUCHOS, en este caso un alumno determinado.

Además, en nuestro controlador `RelacionController` vamos a escribir el código necesario para que nos devuelva los datos relacionados con el alumno con id `1` y la materia con id `2`.

=== "web.php"

    ``` php
    <?php

      use App\Http\Controllers\RelacionController;
      use Illuminate\Support\Facades\Route;

      Route::get('muchos', [ RelacionController::class, 'index' ]);
    ```

=== "RelacionController.php"

    ``` php
    <?php

      namespace App\Http\Controllers;

      use App\Models\Alumno;
      use App\Models\Materia;
      use Illuminate\Http\Request;

      class RelacionController extends Controller
      {
        public function index() {
          $alumno = Alumno::find(1);
          $materia = Materia::find(2);

          return view('muchos', compact('alumno', 'materia'));
        }
      }
    ```

<span class="success">**RELLENANDO LA BASE DE DATOS**</span>

Necesitamos meter algunos registros en nuestra base de datos, por tanto, vamos a crear varios datos en nuestro sistema con las siguientes sentencias SQL.

=== "Tabla Alumnos"

    ``` sql
      INSERT INTO alumnos (`nombre`) VALUES
      ('Antonio'),
      ('Laura'),
      ('Marta'),
      ('Pedro');
    ```

=== "Tabla Materias"

    ``` sql
      INSERT INTO materias (`nombre`) VALUES
      ('Programacion'),
      ('Interfaces'),
      ('JavaScript'),
      ('Sistemas');
    ```

=== "Tabla Alumno_Materia"

    ``` sql
      INSERT INTO alumno_materia (`alumno_id`, `materia_id`) VALUES
      (1, 2),
      (1, 4),
      (3, 2),
      (3, 1),
      (2, 3),
      (2, 4),
      (4, 4),
      (4, 1);
    ```

<span class="success">**CREANDO LA VISTA CON LOS DATOS**</span>

El último paso que vamos a hacer es, listar los datos relacionados en una vista o plantilla `Blade` sencilla. Para ello nos creamos el archivo `muchos.blade.php` ya que es el nombre que hemos puesto en nuestro archivo de rutas.

=== "Alumnos que cursan materias"

    ``` html
    <div class="row justify-content-center">
      <div class="col-auto">
        <h3>Alumno {{ $alumno -> nombre }} está cursando las materias</h3>

        <table class="table table-striped table-hover">
          <thead class="bg-primary text-white">
            <th>MATERIAS</th>
          </thead>

          <tbody>
            @foreach ($alumno -> materias as $registro)
              <tr>
                <td>
                    {{ $registro -> nombre }}
                 </td>
              </tr>
            @endforeach
          </tbody>
        </table>
      </div>
    </div>
    ```

=== "Materias cursadas por alumnos"

    ``` html
    <div class="row justify-content-center">
      <div class="col-auto">
        <h3>La materia {{ $materia -> nombre }} la están cursando los alumnos</h3>

        <table class="table table-striped table-hover">
          <thead class="bg-primary text-white">
            <th>ALUMNOS</th>
          </thead>

          <tbody>
            @foreach ($materia -> alumnos as $registro)
              <tr>
                <td>
                  {{ $registro -> nombre }}
                </td>
              </tr>
            @endforeach
          </tbody>
        </table>
      </div>
    </div>
    ```
---

## Actividades

801. Crear el proyecto CholloSevero:

  - Crea un nuevo repositorio para el proyecto
  - Configura el `.gitignore` para no incluir en el repo los siguientes archivos y carpetas:
      - carpeta `vendor`
      - archivos `.env` y cualquier archivo que empiece por `.` excepto el `.gitignore`
  - La página principal del sitio debe ser un listado con todos los chollos disponibles
  - Configura la base de datos con Eloquent, olvídate de usar la consola mysql
  - Crea una vista para las siguientes acciones:
      - Crear un chollo
      - Editar un chollo
  - La tabla Chollo debe contener las siguientes columnas:
      - `id` único y autoincremental
      - `titulo` un título para el chollo
      - `descripcion` descripcion del chollo
      - `url` un campo para introducir la URL externa del chollo
      - `categoria` albergará la categoría de los chollos
      - `puntuacion` un número entero que indique la puntuación del chollo
      - `precio` para albergar el precio del chollo
      - `precio_descuento` para albergar el nuevo precio
      - `disponible` de tipo boolean
  - Por lo menos, el sitio debe contener un `controlador` de Laravel; puedes crear tantos como creas necesarios pero mínimo debe haber uno.

`::: Elementos estáticos`
Como ya hemos visto, hay ciertos elementos que siempre se muestran en todas las vistas del sitio web. A continuación se listan los elementos que deben estar si o si en todas las plantillas que creéis.

  - Logo del sitio y el título `Chollo ░▒▓ Severo`
  - `Inicio` | `Nuevos` | `Destacados`
  - Un footer con vuestro nombre y algún dato copyright del tipo `©CholloSevero 2022` donde el año debe ser calculado a través de la fecha del servidor.

`::: Pagína principal` <br>
Además del listado de todos los chollos de la base de datos debe contener el menú de navegación:

  - Cada chollo debe ser accesible desde este listado
  - Cada chollo debe contener una imagen que estará guardada en `public/img`
  - Cada chollo debe contener sus botones de `editar` y `borrar` que haga las funciones que tocan. Puedes utilizar iconos para cada uno de los botones.
  - El nombre de las imágenes debe estar compuesta por la siguiente fórmula `idChollo`-chollo-severo.`extension`
      - Por ejemplo: 25-chollo-severo.jpg
  - La imagen del chollo no se sube a través del formulario, la pones directamente en la carpeta. Si te animas a subirlo a través del formulario, puedes hacerlo.

`::: Página de Chollo` <br>
Cuando pinchemos en uno de los chollos del listados debemos ser redireccionados a esta vista donde podremos ver toda la información del tabla chollo. Puedes maquetarla como quieras e incluso puedes basarte en la web de [Chollo Metro](https://www.chollometro.com). El campo `disponible` no es necesario que lo muestres en esta vista

`::: Página de Crear un chollo` <br>
Un formulario con los campos necesarios para poder crear un chollo nuevo. Además, debes tener en cuenta que tienes que validar los campos, de tal manera que no se pueda enviar el formulario si se ha dejado algún campo en blanco; dichas validaciones, además de añadir la propiedad `required` de HTML5 debes hacerlo con Laravel.

En caso de que haya habido algún error en el formulario debes mostrar un mensaje en la parte de arriba del mismo con el mensaje de error (por ejemplo, si el campo está vacío).

`::: Página de Editar un chollo` <br>
Muy parecida a la de Crear un chollo pero que puedas editar un Chollo en función de su `id`. Acuérdate que no puedes dejar ningún campo vacío, para ello has de utilizar las validaciones de Laravel.

`░▒▓ COSAS A TENER EN CUENTA ░▒▓` <br>
- Tienes que usar `Bootstrap` o `Material Design`, aunque si lo prefieres puedes hacer tus propios archivos `.css`

- Los mensajes de error o de información deben estar estilizados para que el usuario pueda verlos con facilidad

- Los elementos estáticos deben estar presentes en todas las vistas; incluidas las de editar y crear.

- Las plantillas que formen parte de otra ya creada deben estar en una carpeta con el nombre de la plantilla madre, como hicimos con el ejercicio de `Notas`:

<div class="center img-large">
    <img src="imagenes/07/estructura-ficheros.png">
</div>

- Ve haciendo commits en función de las tareas que vayas acabando o que veas que el commit tiene sentido. No es buena práctica subir los camios de un archivo y el siguiente commit volver a subir más cambios del mismo archivo (a no ser que nos hayamos saltado o equivocado en algo).

- El proyecto es individual y después se presentará, uno por uno al profesor para que evalúe todos los aspectos del mismo. Se harán preguntas de cómo se ha hecho cierta cosa o por qué se ha determinado cierto flujo de trabajo así que, <span class="alert">***no os copiéis porque se evalúa también la presentación del proyecto***</span>
