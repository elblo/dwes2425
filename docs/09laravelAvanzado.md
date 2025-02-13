# Uso avanzado de Laravel

> Duración estimada: 10 sesiones

## 9.1 Manejo de ficheros

Laravel proporciona una API sencilla y potente para trabajar con ficheros mediante el uso del sistema de almacenamiento basado en **Flysystem**. Esto permite interactuar con el sistema de archivos local y servicios en la nube como Amazon S3 o Dropbox de manera uniforme.

### Configuración del almacenamiento

Laravel usa la configuración del sistema de archivos en `config/filesystems.php`. El driver por defecto es `local`, pero se pueden configurar otros como `s3` o `public`.

```php
<?php
// Configuración del disco local privado, público y o s3
return [
    'default' => env('FILESYSTEM_DISK', 'local'), // local | public | s3

       'disks' => [

          'local' => [
              'driver' => 'local',
              'root' => storage_path('app/private'),
              'serve' => true,
              'throw' => false,
              'report' => false,
          ],

          'public' => [
              'driver' => 'local',
              'root' => storage_path('app/public'),
              'url' => env('APP_URL').'/storage',
              'visibility' => 'public',
              'throw' => false,
              'report' => false,
          ],

          's3' => [
              'driver' => 's3',
              'key' => env('AWS_ACCESS_KEY_ID'), // Definidas en .env
              'secret' => env('AWS_SECRET_ACCESS_KEY'),
              'region' => env('AWS_DEFAULT_REGION'),
              'bucket' => env('AWS_BUCKET'),
              'url' => env('AWS_URL'),
              'endpoint' => env('AWS_ENDPOINT'),
              'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
              'throw' => false,
              'report' => false,
          ],
      ],
];
```

### Almacenar archivos

Para subir y guardar archivos en Laravel, se utiliza la clase `Illuminate\Support\Facades\Storage`. A sus métodos se les pasa el *path* y *nombre* del archivo con el que operar. Hay que tener en cuenta que el *path* se indica a partir del disco configurado en el punto anterior, así que, si por ejemplo queremos almacenar un archivo en `/storage/app/private/archivos` únicamente habría que indicar como path `archivos`.

```php
<?php
// Función 'storage' en controlador que recibe los datos del formulario
use Illuminate\Support\Facades\Storage;

// 'imagen' es el atributo name del input type file del formulario
$request->validate([
    'imagen' => 'required|image|mimes:jpeg,png,jpg,gif|max:2048',
]);

// Opción 1.1 Guardar un archivo en el disco 'local' ('storage/app/private/archivos/')
$path = Storage::put('archivos', $request->file('imagen')); // Storage siempre a `local` por defecto 
$path = Storage::disk('public')->put('archivos', $request->file('imagen')); // Elegir el disco al que guardar 

// Opción 1.2: Guardar archivo con nombre específico
$path = Storage::putFileAs('archivos', $request->file('imagen'), 'mi_imagen.jpg');

 // Opción 2.1: Guardar desde $request
$path = $request->file('imagen')->store('archivos'); // 'local' por defecto --> Guarda en 'storage/app/private/archivos/'
$path = $request->file('imagen')->store('archivos', 'public'); // 'public' --> Guarda en 'storage/app/public/archivos/'

// Opción 2.2: Guardar desde $request con nombre específico
$path = $request->file('archivo')->storeAs('archivos', 'nuevonombre.jpg', 'public');

return back()->with('success', 'Imagen subida con éxito a la ruta:' . $path); // Vuelve atrás pasando la variable 'success' por la sesión  
```

### Obtener archivos

Cuando accedamos al **storage** siempre va a buscar a partir del directorio correspondiente al disco configurado en filesystem.

```php
<?php
// Obtener el contenido de un archivo (con ficheros de texto, no imágenes)
$contenido = Storage::get('archivos/mi_archivo.pdf');
echo $contenido;

// Verificar si un archivo existe del disco 'local' o del 'public'
if (Storage::exists('archivos/mi_archivo.pdf')) echo "El archivo existe.";
if (Storage::disk('public')->exists('archivos/mi_archivo.pdf')) echo "El archivo existe.";

// Descargar un archivo
return Storage::download('archivos/mi_archivo.pdf');
```

### Eliminar archivos

```php
<?php
// Eliminar un archivo del disco 'local' o del 'public'
Storage::delete('archivos/mi_archivo.pdf');
Storage::disk('public')->delete('archivos/mi_archivo.pdf');
// Eliminar múltiples archivos
Storage::delete(['archivos/archivo1.pdf', 'archivos/archivo2.pdf']);
```

### Listar archivos y directorios

```php
<?php
// Obtener todos los archivos de un directorio del disco 'local' o del 'public'
$archivos = Storage::files('archivos');
$archivos = Storage::disk('public')->files('archivos');
print_r($archivos);
// Obtener todos los archivos de un directorio incluyendo subdirectorios
$archivos = Storage::allFiles('archivos');
print_r($archivos);
// Obtener sólo los directorios
$directorios = Storage::directories('archivos');
print_r($directorios);
```

### Crear enlace simbólico

Para acceder a archivos desde el navegador mediante su ruta y poder incluirlos en las vistas, es necesario crear un enlace simbólico en `/public` que apunte a  `/storage/app/public` (o donde mande el disco configurado). Para ello, desde la raíz del proyecto ejecuta:

```console
php artisan storage:link
```

En `/public` se habrá creado el enlace símbolico **storage**, que en la práctica es como si tuviésemos ahí los archivos. 

### Ejemplo subir archivo

En la vista:

```html
<form action="{{ route('subir.archivo') }}" method="POST" enctype="multipart/form-data">
    @csrf
    <input type="file" name="archivo">
    <button type="submit">Subir</button>
</form>
```

En el controlador:

```php
<?php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

public function subirArchivo(Request $request) {
    $request->validate([
        'archivo' => 'required|file|max:2048',
    ]);
    
    $path = $request->file('archivo')->store('archivos');
    return "Archivo subido a: " . $path;
}
```

### Ejemplo archivos en vistas

En el controlador:

```php
<?php
public function muestraArchivos(){
      $paths = Storage::disk('public')->allFiles('archivos');
      return view('archivos.muestra', compact('paths')); 
  }
```

En la vista:

```html
<div>
  <h1>Listado de imágenes</h1>
  @foreach ($paths as $path)
      <p><img src="{{ Storage::url($path) }}" /></p>
  @endforeach
</div>
```

## 9.2 Request y Response

Las peticiones y respuestas permiten interactuar con la solicitud HTTP que llega y que se devuelve, por lo que se suelen utilizar en alguna función del controlador al que dirige una ruta determinada. Más info en la [documentación oficial](https://laravel.com/docs/11.x/requests).

### Request

La clase `Illuminate\Http\Request` de Laravel proporciona una forma orientada a objetos de interactuar con la solicitud HTTP actual que maneja la aplicación, así como también de recuperar la entrada, las cookies y los archivos que se enviaron con la solicitud.

#### Acceso a los datos de la petición

Se pueden obtener los elementos enviados a la petición de diferentes formas. Recuerda que se recuperan por el campo `name` que hayas especificado en el formulario.

```php
<?php
$input = $request->all(); //Acceder a todos los inputs
$name = $request->input('name'); //Obtener un input específico
$name = $request->name; //Obtener un input específico
$age = $request->input('age', 18); //Especificar valores por defecto
$id = $request->route('id'); //Acceder a parámetros de ruta
```

También es posible comprobar si en la petición se han recibido ciertos elementos, si vienen rellenos, excluirlos...

```php
<?php
if ($request->has('email')) {
    // Input 'email' se ha recibido
}
if ($request->filled('name')) {
    // Input 'name' no está vacío
}
$filtered = $request->only(['name', 'email']); // Filtrar inputs específicos
$excluded = $request->except(['password']); // Excluir ciertos inputs
```

#### Tratamiento de archivos

Los archivos se tratan de forma similar.

```php
<?php
// Comprobar si se ha recibido el archivo con name 'photo'
if ($request->hasFile('photo')) { 
    $file = $request->file('photo');
}
// Almacenar el archivo en el storage configurado
$path = $request->file('photo')->store('photos'); 
```

### Response

Una instancia de Response hereda de la clase `Symfony\Component\HttpFoundation\Response` y proporciona una variedad de métodos para personalizar el código de estado HTTP y los encabezados de la respuesta.

#### Crear respuestas

```php
<?php
// Respuesta básica
return response('Hello World', 200); 
// Respuesta en formato JSON
return response()->json([
    'name' => 'John',
    'status' => 'success'
]); 
 // Redirección
return redirect('dashboard');
// Redicrección pasando la variable 'status'
return redirect('login')->with('status', 'Sesión iniciada'); 
```

#### Manipular cabeceras

```php
<?php
// Respuesta añadiendo 1 cabecera
return response('Hello')->header('Content-Type', 'text/plain');
// Respuesta añadiendo múltiples cabeceras
return response('Hello')
  ->header('Content-Type', 'application/json')
  ->header('Cache-Control', 'no-cache');
```

#### Respuestas de archivos

```php
<?php
return response()->download($pathToFile); // Descarga archivo
return response()->file($pathToFile); // Mostrar archivo
```

## 9.3 Eloquent: Relaciones

A través de Eloquent vamos a poder gestionar las relaciones entre nuestras tablas de la base de datos de una manera muy sencilla y sin sentencias SQL. Más info sobre relaciones, como siempre, en la [documentación oficial](https://laravel.com/docs/11.x/eloquent-relationships).

### Relaciones en BBDD

En una base de datos relacional, las relaciones son las conexiones entre las tablas, que se establecen a través de las claves primarias y foráneas. Para obtener datos de varias tablas, podemos hacerlo de 2 formas:

- *JOIN*: Unir las tablas a través de las claves primarias y foráneas.
- *Relaciones*: Definir las relaciones en los modelos, y acceder a los datos a través de las relaciones.

Las relaciones que existen en una base de datos relacional son:

- **Uno a Uno**: Un registro de una tabla se relaciona con un único registro de otra tabla.
- **Uno a Muchos**: Un registro de una tabla se relaciona con varios registros de otra tabla.
- **Muchos a Muchos**: Varios registros de una tabla se relacionan con varios registros de otra tabla.
- **Uno a Uno Polimórfica**: Un registro de una tabla puede relacionarse con un único registro de varias tablas.
- **Uno a Muchos Polimórfica**: Un registro de una tabla puede relacionarse con varios registros de varias tablas.
- **Muchos a Muchos Polimórfica**: Varios registros de una tabla pueden relacionarse con varios registros de varias tablas.

Nota: Relaciones polimórficas son aquellas en las que una tabla se relaciona con varias tablas.

### Relaciones en Eloquent

Eloquent nos permite definir las relaciones entre los modelos, y acceder a los datos a través de ellas. Tipos:

- **Uno a Uno**: `hasOne()`, `belongsTo()`
- **Uno a Muchos**: `hasMany()`, `belongsTo()`
- **Muchos a Muchos**: `belongsToMany()`
- **Uno a Uno Polimórfica**: `morphOne()`, `morphTo()`
- **Uno a Muchos Polimórfica**: `morphMany()`, `morphTo()`
- **Muchos a Muchos Polimórfica**: `morphToMany()`, `morphedByMany()`

El método a utilizar `hasOne()` o `belongsTo()` y similares dependerá de la tabla en la que se encuentre la clave foránea. Por ejemplo, si en una aplicación de Notas, la tabla notas tiene una clave foránea user_id, está utilizará belongsTo() y la tabla usuarios, hasOne(), hasMany() o lo que corresponda.

### Relación Uno a uno (1 a 1)

Para crear este tipo de relaciones en Eloquent y Laravel, debemos tener creadas las tablas que vayamos a relacionar y establecer la relación entre ellas a través del método `hasOne()` y `belongsTo()`.

Supongamos que tenemos una tablas `usuario` que está relacionada con la tabla `telefono`.

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Usuario extends Model
{
  // Obtener el Teléfono asocioado con el Usuario
  public function telefono()
  {
      return $this->hasOne(Telefono::class);
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

### Relación Uno a Uno ***INVERSA***

Ahora que podemos acceder al modelo teléfono desde el modelo usuario, vamos a ver cómo hacerlo de manera inversa, es decir, cómo acceder desde el modelo `telefono` al modelo `usuario` mediante el método `belongsTo()`.

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Telefono extends Model
{
  public function usuario()
  {
      return $this->belongsTo(Usuario::class);
  }
}
```

Al llamar el método de `usuario`, Eloquent intentará encontrar un modelo Usuario que tenga un `id` que coincida con la columna de `usuario_id` en el modelo de `telefono`.

Eloquent determina el nombre de la clave externa examinando el nombre del método de relación y agregando el sufijo `_id` al nombre del método. Entonces, asume que el modelo `Telefono` tiene una columna `usuario_id`. Sin embargo, si no se llama de esa manera, puedes pasarle como argumento el nombre de la clave.

```php
<?php
public function usuario()
{
    return $this->belongsTo(Usuario::class, 'clave_ajena');
}
```

### Relación Uno a Muchos (1 a M)

Por ejemplo, las entradas de un blog o un post tienen muchos comentarios.

Para empezar, ya sabemos que debemos crear ambas clases del modelo y en este caso, usaremos el método `hasMany()` para obtener los datos relacionados con ese post o entrada en el blog.

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
  public function comentarios()
  {
      return $this->hasMany(Comentario::class);
  }
}
```

Cuidado con las claves ajenas, que aquí pasa lo mismo... Eloquent establece por defecto el sufijo `_id` por lo tanto, en este ejemplo buscaría por `post_id`. Si nuestra clave ajena tiene otro nombre, se lo pasamos por parámetro en el método `hasMany` como hacíamos más arriba.

Hay que tener en cuenta que `hasMany` devuelve un array de elemento.

```php
<?php
use App\Models\Post;

$comentarios = Post::find(1)->comentarios;

foreach ($comentarios as $comentario) {
    // Lo que sea que hagamos con esos datos o pasar el array directamente a la vista
}
```

Además, como todas las relaciones son sentencias SQL, podemos anidar varios filtros en función de lo que queramos sacar.

```php
$comentario = Post::find(1)->comentarios()
    ->where('titulo', 'lo que sea')
    ->first();
```

### Relación Uno a Muchos ***INVERSA***

Ahora que podemos acceder a todos los comentarios de una publicación, definamos una relación para permitir que un comentario acceda a su publicación principal.

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Comentario extends Model
{
  public function post()
  {
      return $this->belongsTo(Post::class);
  }
}
```

Y ahora, a través de la propiedad de relación dinámica...

```php
<?php
use App\Models\Comentario;

$comentario = Comentario::find(1);

return $comentario->post->titulo;
```

Pasaría lo mismo con el nombre de la clave ajena, si no se llama de la misma manera que Eloquent establece con el sufijo `_id` podemos pasarle como parámetro el nombre de la clave donde debe buscar.

### Relación Muchos a Muchos (MM a MM)

Este tipo de relaciones son las más complicadas. Por ejemplo, en un blog como Wordpress, un usuario puede tener muchos roles (lector, autor, administrador) pero un rol pueden tenerlo varios usuarios, es decir, puede haber muchos usuarios administradores, otros lectores y demás.

Para realizar este tipo de relaciones necesitaríamos 3 tablas diferentes.

  - usuarios [id, nombre]
  - roles [id, nombre]
  - rol_usuario [usuario_id, rol_id] (Tabla Pivote)

Lo primero de todo, vamos a crear las tablas con sus modelos <span class="alert">***a excepción de la tabla pivote rol_usuario***</span> que <span class="warning">***sólo crearemos la tabla, sin su modelo***</span>

```console
php artisan make:migration create_rol_usuario_table --create=rol_usuario
```

Y la estructura de dicha tabla quedaría:

```php
<?php

public function up()
{
    Schema::create('rol_usuario', function (Blueprint $table) {
        $table->bigIncrements('id'); // creates an auto-incrementing UNSIGNED BIGINT (primary key) equivalent column
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
        return $this->belongsToMany(Rol::class);
    }
}
```

Como 2º parámetro de `belongsToMany` se puede indicar la tabla intermedia de la relación `rol_usuario`. Útil, si se utiliza un nombre que Laravel no puede deducir a partir de los modelos.

Una vez que tengamos las relaciones definidas, accederemos a ellas mediante las propiedades dinámicas de `roles`.

```php
<?php

use App\Models\Usuario;

$usuario = Usuario::find(1);

foreach ($usuario->roles as $rol) {
    // nuestro código
}
```

Acordaros que podemos encadenar comandos sql a través de los métodos de Eloquent

```php
<?php
$roles = Usuario::find(1)->roles()->orderBy('nombre')->get();
```

### Relación Muchos a Muchos ***INVERSA***

Para definir la parte "inversa" de una relación de muchos a muchos, debemos establecer un método en el modelo relacionado que también devuelva el resultado del método `belongsToMany`. En este caso, se hace exactamente igual en ambas partes de la relación. Según el ejemplo que estamos siguiendo:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Rol extends Model
{
  public function usuarios()
  {
      return $this->belongsToMany(Usuario::class);
  }
}
```

### Relaciones transitivas

Las relaciones transitivas **Has One Through** y **Has Many Through** nos permiten una forma de acceder a tablas lejanas, que no están directamente relacionadas entre sí. Por ejemplo, Un Editor tiene muchos Post, y un Post tiene muchos Comment. Si queremos acceder a los Comment de un Editor, podemos hacerlo a través de la relación Has Many Through.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Editor extends Model
{
  public function comments()
  {
      return $this->hasManyThrough(Comment::class, Post::Class);
  }
}
```

### Ejemplo completo

Vamos a hacer un ejemplo con una APP que gestiones alumnos y asignaturas, de tal manera que MUCHOS ALUMNOS pueden cursar MUCHAS ASIGNATURAS

¿Qué necesitamos para este ejemplo?

  - 2 **modelos**:  `Alumno` y `Materia`.
  - 3 migraciones para crear las tablas: `alumnos`, `materias` y `alumno_materia` (tabla pivote).
  - Modificar los archivos de las migraciones `create_alumnos_table` y `create_materias_table` añadiendo los campos que se necesiten.
  - Ejecutar las migraciones.
  - Rellenar la base de datos con alguna información de prueba.
  - Método dentro de `Alumno` para crear la relación Alumno->Materia.
  - Método dentro de `Materia` para crear la relación Materia->Alumno. 
  - Crear las rutas necesarias para ambas peticiones.
  - Crear las funciones necesarias en los controladores que redirijan a la vistas.
  - Crear las vistas.

#### 1. Crear los modelos
```console
php artisan make:model Alumno
php artisan make:model Materia
```

#### 2. Crear las 3 migraciones

```console
php artisan make:migration create_alumnos_table
php artisan make:migration create_materias_table
php artisan make:migration create_alumno_materia_table
```

#### 3. Modificar las migraciones

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

#### 4. Ejecutar las migraciones

Lanzar las migraciones para que se creen las tablas en la BDD. Rellenar con alguna información de prueba.

``` console
php artisan migrate
```

#### 5. Rellenar la BDD con datos de prueba

Crear varios datos de prueba insertando por ejemplo, los siguientes registros mediante sentencias SQL.

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

#### 6. Relación Alumno<->Materia

En otro tipo de relaciones se podía indicar la clave ajena, en esta se indica la tabla intermedia de la relación `alumno_materia`.

=== "Alumno.php"

    ``` php
    <?php
    public function materias() {
      return $this->belongsToMany(Materia::class, 'alumno_materia');
    }
    ```

=== "Materia.php"

    ``` php
    <?php
    public function alumnos() {
      return $this->belongsToMany(Alumno::class, 'alumno_materia');
    }
    ```

#### 7. Crear las rutas

Vamos a crear las rutas para que dado el id de un alumno pasado por parámetro, devuelva un listado de sus asignaturas y viceversa.

=== "web.php"

    ``` php
    <?php
      use App\Http\Controllers\RelacionController;
      use Illuminate\Support\Facades\Route;

      Route::get('alumnos/{id}/materias', [AlumnoController::class, 'materias']);
    ```

Haz tú la ruta, para que dado el id de una asignatura pasada por parámetro, devuelva un listado de los alumnos que la cursan.

#### 8. Crear el controlador

Necesitamos controladores para redireccionar las rutas a las vistas que nosotros queramos, para ello empezamos creando el controlador `AlumnoController`.

```console
php artisan make:controller AlumnoController
```

=== "AlumnoController.php"

    ``` php
    <?php
      namespace App\Http\Controllers;

      use App\Models\Alumno;
      use Illuminate\Http\Request;

      class AlumnoController extends Controller
      {
        public function materias(string $id) {
          $materias = Alumno::find($id)->materias;

          return view('alumnos.materias', compact('materias'));
        }
      }
    ```
  
Haz tú ahora el controlador `MateriaController` con su función correspondiente para devolver a los alumnos que cursen una materia determinada.

#### 9. Crear la vista

Vamos a crar la vista que liste las materias de un usuario. Para ello creamos el archivo `alumnos/materias.blade.php`.

    ``` html
    <div class="row justify-content-center">
      <div class="col-auto">
        <h3>Las materias que está cursando el alumno son:</h3>

        <table class="table table-striped table-hover">
          <thead class="bg-primary text-white">
            <th>MATERIAS</th>
          </thead>

          <tbody>
            @foreach ($materias as $materia)
              <tr>
                <td>
                    {{ $materia->nombre }}
                 </td>
              </tr>
            @endforeach
          </tbody>
        </table>
      </div>
    </div>
    ```

¿Y si quisiéramos además dar información del propio alumno? Como su nombre por ejemplo, ¿qué más tendrías que pasar a la vista?

Haz tú ahora la vista correspondiente a los alumnos que cursan una determinada materia en el archivo `materias/alumnos.blade.php`.

### Añadir/eliminar elementos a la tabla pivote

Laravel proporciona una serie de métodos para añadir/eliminar elementos a la tabla pivote en las relaciones de Muchos a Muchos. Estos son:

- `attach()`: Añade a la tabla pivote los elementos por su id.
- `detach()`: Elimina de la tabla pivote los elementos por su id.
- `toggle()`: Añade un elemento si no existe, y lo elimina si ya existe.
- `sync()`: Deja en la tabla pivote sólo los elementos que se le pasen.

Los métodos anteriores admiten un único id, un array de ids o el propio objeto. Ejemplos:

```php
<?php
  use App\Models\Alumno;
  use App\Models\Materia;

  // Recuperar alumno con id 1
  $alumno = Alumno::find(1);

  // Añadirle materias por sus ids
  $alumno->materias()->attach([1, 2, 3]);
  // Quitarle materias por sus ids
  $alumno->materias()->detach(2);
  // Elimina el 1 (ya existía) y añade el 2 (no existía)
  $alumno->materias()->toggle([1, 2]);
  // Sincroniza con los elementos pasados (sólo deja estos en la tabla pivote)
  $alumno->materias()->sync([1, 3]);

  // También se pueden releacionar mediante el propio objeto
  $materia = Materia::find(1);
  $alumno->materias()->attach($materia); // En este caso da error porque ya tiene dicha materia (1)
```

## 9.4 Mutadores y accesores

Los **mutatores** permiten transformar datos antes de guardarlos y los **accesores** los transforman al recuperarlos.

Ejemplo sobre el atributo password:

- El mutador (set) encripta la contraseña con bcrypt().
- El accesor (get) devuelve "********" para ocultar la contraseña al acceder al modelo.

```php
<?php
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected function password(): Attribute
    {
        return Attribute::make(
            get: fn ($value) => '********', // Oculta la contraseña al acceder
            set: fn ($value) => bcrypt($value), // Encripta al guardar
        );
    }
}
```

Uso:

```php
<?php
$user = new User();

// Establecemos la contraseña (Laravel la encripta automáticamente)
$user->password = 'mi_contraseña_segura';
echo $user->password; // Salida: ******** (accesor oculta el valor)

$user->save();

// Verificamos en la base de datos
echo $user->getAttributes()['password']; // Salida: $2y$10$...
```

Más info en [mutadores y accesores](https://laravel.com/docs/11.x/eloquent-mutators#defining-an-accessor).

## 9.5 Seeders y factorías

Los seeders y factorías permiten generar datos de prueba de forma fácil y rápida, útiles durante el desarrollo para simular datos iniciales en una aplicación.

### Seeders

Los seeders son clases especiales que permiten "sembrar" datos en la base de datos.

#### Crear un seeder

Mediante el siguiente comando crearmos un seeder con nombre `BooksSeeder` en el directorio *database/seeders*:

```console
php artisan make:seeder BooksSeeder
```

En el método `run` añadimos los elementos que queremos crear:

```php
<?php
class BooksSeeder extends Seeder
{
    public function run()
    {
        // Ejemplo: Crear un libro
        $book = new Book();
        $book->title = "Laravel for Beginners";
        $book->author = "John Doe";
        $book->save();
    }
}
```

#### Añadir el seeder a DatabaseSeeder

El seeder recién creado hay que añadirlo al `DatabaseSeeder`. Por ejemplo:

```php
<?php
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            BooksSeeder::class,
            AuthorsSeeder::class,
            // ...
        ]);
    }
}
```

#### Ejecutar el seeder

Podemos hacer varias cosas, según nos interese:

- `php artisan db:seed` : Ejecutar todos los seeders.
- `php artisan db:seed --class=BooksSeeder` : Ejecutar un seeder escífico.
- `php artisan migrate:fresh --seed` : Reiniciar las migraciones y ejecutar todos los seeders.

### Factorías

Las factorías permiten crear grandes cantidades de datos "fake" de forma rápida y dinámica. Utiliza la librería [Faker](https://fakerphp.org/) para ello.

#### Crear una factoría

Mediante el siguiente comando creamos una factoría con nombre `AuthorFactory` asociada al modelo `Author` en el directorio *database/factories*:

```console
php artisan make:factory AuthorFactory -m Author
```

En el método `definition` devolvemos un array asociativo con los campos del objeto que queremos crear.

```php
<?php
namespace Database\Factories;

use App\Models\Author;
use Illuminate\Database\Eloquent\Factories\Factory;

class AuthorFactory extends Factory
{
    // Asociación de la factoría con el modelo (en este caso se podría omitir porque Laravel la deduce por el nombre)
    protected $model = Author::class; 

    public function definition()
    {
        return [
            'name' => $this->faker->name,
            'birth_year' => $this->faker->year,
        ];
    }
}
```

#### Asociar factoría al modelo

Hay que asociar la factoría con el modelo tanto en la factoría como en el modelo.

- En la **parte de la factoría** se realiza con convención, por lo que si la factoría se llama AuthorFactory, el modelo asociado será Author. Y en caso de que queramos indicar esta asociación de forma explícita, lo podemos hacer a través de la propiedad `$model` como en el ejemplo anterior.
- En la **parte del modelo**, si lo hemos creamos a mano, hay que indicar que usa una factoría mediante el [trait](https://www.php.net/manual/en/language.oop5.traits.php) `HasFactory`:

```php
<?php
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Author extends Model
{
    use HasFactory;
    //...  
}
```

#### Utilizar una factoría

```php
<?php
// Donde nos interese, por ejemplo en el controlador, modelo...
use App\Models\Author;

// Crear un autor
Author::factory()->create();

// Crear 10 autores
Author::factory()->count(10)->create();
```

#### Integración con seeders

Combinar factorías con seeders se pueden crear datos dinámicos de forma sencilla. No olvidar incluir los seeders en `DatabaseSeeder`.

```php
<?php
class AuthorsSeeder extends Seeder
{
    public function run()
    {
        Author::factory()->count(10)->create();
    }
}

// Utilizar factorías con datos relacionados
class BooksSeeder extends Seeder
{
    public function run()
    {
        $authors = Author::factory()->count(5)->create();

        $authors->each(function ($author) {
            Book::factory()->count(2)->create(['author_id' => $author->id]);
        });
    }
}
```

## Actividades

A continuación, vas a realizar una serie de ejercicios sobre cada uno de los apartados vistos en el tema. Puedes crear un proyecto nuevo o reutilizar uno existente.

### Manejo de ficheros

Para practicar con los ficheros vas a crear una galería de imágenes con posibilidad de subir nuevas, eliminar y acceder a su vista en detalle. También vas a implementar un "Mini Drive" para gestionar archivos. En ambos casos, vas a trabajar sin modelos para centrarte exclusivamente en el manejo de ficheros, pero en una app real, además sería recomendable trabajar con una BDD que almacene la información necesaria de los archivos.

901. **Formulario de subida**: Crea un formulario para subir una imagen. La ruta que lleva al formulario por GET será `imagen/create` y la vista del mismo será `imagen/create.blade.php`. El formulario se enviará por POST a la ruta `imagen/storage`. Además del propio input de la imagen, el formulario tendrá un radio button para seleccionar si la imagen se almacenará de forma privada o pública en el storage.

902. **Almacenar archivos**: Crea la función correspondiente en el controlador para recibir la imagen del formulario anterior validando que sea del tipo imagen, requerida y con un tamaño máximo de 2MB. Y según la opción del formulario, almacena la imagen de forma privada o pública en el storage. Después redirecciona a la página anterior enviando un mensaje del tipo "Imagen NOMBRE almacenada correctamente en el storage privado|público" que mostrarás justo encima del formulario.

903. **Mostrar archivos**: Mediante la ruta por GET `imagen` que lleva a la vista `imagen/index.blade.php` muestra una galería con todas las imágenes en miniatura. Puedes utilizar flexbox o grid layout para posicionarlas unas al lado de las otras. **Importante**: No olvides crear el enlace simbólico para poder acceder a las imágenes. Crea un enlace "Subir imagen" que lleve al formulario del primer punto y en dicho formulario, un enlace para volver aquí, al listado.

904. **Mostrar imagen completa**: Mediante la ruta por GET `imagen/{name}` que lleva a la vista `imagen/show.blade.php` muestra la vista en tamaño completo de la imagen. Muestra un párrafo con su ruta completa y un enlace para volver al listado.
    
905. **Eliminar archivos**: Mediante la ruta por GET `imagen/{name}/destroy` elimina la imagen que corresponda. A esta ruta podrás llegar mediante un enlace de la vista en detalle de la imagen. Una vez eliminada, se redirige automáticamente al listado enviando con 'with' un mensaje del tipo "Imagen NOMBRE eliminada correctamente del storage público". Si lo prefieres, en lugar del enlace a eliminar, puedes hacerlo mediante un formulario que lleve a una ruta DELETE `imagen/{name}`.

906. **Almacenamiento en S3** (opcional): Con tu cuenta de estudiante de AWS, crea un bucket S3 con acceso público. Configura el proyecto actual para utilizar el disco 's3' por defecto o bien utilízalo de forma explícita en cada interacciíon que realices con el Storage.

907. **Mini Drive**: De forma similar a lo que acabas de hacer con la galería, crea un sistema de almecenamiento de archivos que admita archivos de diferente tipo (imágenes, videos, documentos...). Deberás mostrar un listado con un icono según el tipo de archivo, su nombre, tamaño y opciones (eliminar), un formulario para subirlo con un campo en el que recojas el nombre con el que almacenarlo. Y en vez de la vista en detalle, al pulsar sobre el archivo en la vista del listado, se descargará directamente. Sigue las recomendaciones de los puntos anteriores. Recuerda que cada archivo no va a tener un modelo asociado, por lo que el nombre, tamaño... deberás obtenerlo de la información del propio archivo.

908. **Mini Drive con directorios** (opcional): Investiga cómo crear directorios y mover archivos entre ellos. Ofrece en la interfaz que has creado, las opciones correspondientes para crear un nuevo directorio, para cambiar el nobmre a un archivo (si no existe uno ya con dicho nombre) y moverlo a un directorio determinado. 

### Request y response

En los siguientes ejercicios vas a trabajar con `Request`y `Response` en las funciones de los controladores. El primero ya lo has utilizado para recoger los datos recibidos de un formulario. Vas a repasar su uso y sobre todo, vas a conocer el segundo.

910. **Obtener y validar datos con Request**: Crea un pequeño formulario para recoger "Nombre" y "Email" del usuario. Recoge sus datos en una función del controlador y valídalos. Si pasan la validación, símplemente devuelve un mensaje "Se ha pasado la validación" y si no, captura los errores en la vista del formulario.

911. **Redireccionar la respuesta**: Modifica el ejercicio anterior para redireccionar la respuesta a otra ruta.

912. **Respuesta JSON**: Crea una ruta GET `/api/usuarios` que devuelva un array de usuarios (id, nombre, email) en formato JSON.

913. **Respuesta JSON error**: Crea una ruta GET `/api/error` que devuelva un array (error y mensaje) en formato JSON y además, el código de estado 400 para indicar al cliente que ha enviado una petición inválida.

914. **Modificar cabeceras de la respuesta**: Crea una ruta GET `/archivo/descargar` que devuelva un archivo descargable y modifique los encabezados de la respuesta. En el controlador, usa *response()->download()* para devolver el archivo estableciendo un encabezado personalizado.

915. **Ejercicio completo: API Rest**: Con lo visto hasta ahora, implementa una API RESTful para manejar usuarios:

- **C**: Crear un usuario mediante POST `/api/usuarios`.
- **R**: Listar usuarios con GET `/api/usuarios`.
- **U**: Actualizar un usuario con PUT `/api/usuarios/{id}`.
- **D**: Eliminar un usuario con DELETE `/api/usuarios/{id}`.

Asegúrate de devolver respuestas adecuadas en JSON y manejar los errores correctamente.

### Eloquent: Relaciones

En este apartado vas a crear diferentes relaciones entre modelos.

920. **Relación 1 a 1**. 

- Crea los modelos `Usuario` y `Perfil`. Cada usuario tiene un perfil, y cada perfil pertenece a un único usuario.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `usuarios`: campos `id`, `nombre`, `email`.
  - `perfiles`: campos `id`, `usuario_id`, `telefono`, `direccion`.
- Define la relación en los modelos.
- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta los datos de un usuario y muestra su perfil. Para ello, crea una ruta `usuario/{id}` que redirija a la función `show` del controlador y llame a la vista `usuario.show.blade.php` para los datos del usuario con su perfil.

921. **Relación 1 a Muchos**. 

- Crea los modelos `Categoria` y `Producto`. Cada categoría tiene muchos productos, pero un producto sólo perteneca una determinada categoría.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `categorias`: campos `id`, `nombre`.
  - `productos`: campos `id`, `nombre`, `categoria_id`.
- Define la relación en los modelos.
- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta el nombre de una categoría mostrando sus productos. Para ello, crea una ruta `categoria/{id}` que redirija a la función `show` del controlador y llame a la vista `categoria.show.blade.php`.
- Mediante Eloquent agrega un nuevo producto a la categoría con id pasado por parámetro. Para ello, crea una ruta `categoria/{id}/addproduct/{nombre}` que redirija a la función `addProduct` del controlador y redirija a la ruta `show` anterior que llama a la vista `categoria.show.blade.php`.

922. **Relación Muchos a Muchos**. 

- Crea los modelos `Estudiante` y `Asignatura`. Cada estudiante puede estár matriculado en muchas asignaturas y una asignatura la cursan muchos estudiantes.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `estudiantes`: campos `id`, `nombre`.
  - `asignaturas`: campos `id`, `nombre`.
  - `asignatura_estudiante` (tabla pivote): `estudiante_id`, `asignatura_id`.
- Define la relación en los modelos.
- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta todas las asignaturas de un estudiante por su id. Para ello, crea una ruta `estudiante/{id}` que redirija a la función `show` del controlador y llame a la vista `estudiante.show.blade.php`.
- Consulta ahora todos los estudiantes que cursen una asignatura por su id. Para ello, crea una ruta `asignatura/{id}` que redirija a la función `show` del controlador y llame a la vista `asignatura.show.blade.php`.
- Mediante Eloquent matricula a un estudiante en un curso determinado. Para ello, crea una ruta `estudiante/{id}/matricula/{idCurso}` que redirija a la función `matricula` del controlador y redirija a la ruta `show` de estudiante que llama a la vista `estudiante.show.blade.php`.
- Mediante Eloquent desmatricula a un estudiante en un curso determinado. Para ello, crea una ruta `estudiante/{id}/desmatricula/{idCurso}` que redirija a la función `desmatricula` del controlador y redirija a la ruta `show` de estudiante que llama a la vista `estudiante.show.blade.php`.

923. **Ejercicio completo**: CRUD con varias relaciones y formularios.

- Implementa un sistema de gestión de posts con sus respectivos comentarios mediante los modelos `Autor`, `Post` y `Comentario`.
- Piensa bien las relaciones a utilizar. Un autor puede escribir muchos posts y comentarios. Un post puede tener muchos comentarios. Un comentario sólo pertenece a un post y está escrito por un autor.
- De un *autor* interesa saber su imagen, nombre y email.
- De un *post* interesa saber su título, fecha y descripción.
- De un *comentario* interesa saber su texto y fecha.
- Crea los CRUDs necesarios para cada modelo con sus rutas específicas, controladores, vistas (con formularios)...

### Mutadores y accesores

930. **Formatear nombres y convertir números**: En el modelo `Producto` del ejercicio anterior, crea:

- Un mutador que almacene el nombre en minúsculas y un accesor que los devuelva con la primera letra en mayúscula.
- Un mutador que almacene el precio convertido a céntimos y un accesor que lo devuelva de nuevo en euros.

931. **Slug automático**: Un slug es una versión formateada de un texto, generalmente usada en URLs por gestores de contenidos. Un slug se crea eliminando caracteres especiales, convirtiendo espacios en guiones y pasando a minúsculas el texto. Por ejemplo: "Hola Mundo Laravel" tendría de slug "hola-mundo-laravel".

En el modelo `Post` del ejercicio anterior:

- Crea la migración correspondiente para añadir el campo `slug` de tipo string.
- Investiga cómo usar `Str::slug` para generar slugs.
- Crea un mutador que convierta el título a slug y lo almacene en el campo `slug`.

932. **Formatear fechas de creación**: En el modelo `Estudiante` del ejercicio anterior:

- Investiga cómo usar la biblioteca `Carbon` para trabajar con fechas incluida en Laravel.
- Crea un accesor que formatee la fecha de creación (created_at) en formato "d/m/Y - H:i".

### Seeders y factories

940. **Seeder básico**: Crea un nuevo modelo `Usuario` con campos `nombre`, `email` y `password` (en su migración) y crea un seeder `UsuarioSeeder` que inserte 3 usuarios de prueba en la base de datos.

941. **Factoría con seeder**: Crea una factoría `UsuarioFactory` con datos fake y en el seeder `UsuarioSeeder` crea 10 usuarios mediante la factoría.

942. **Seeders con modelos relacionados**: Crea el modelo `Publicacion` con los campos `titulo`, `contenido` y `usuario_id` (en su migración) y modifica los modelos para que un usuario se relacione con muchas publicaciones. Crea las factorías `UsuarioFactory` (ya la tienes) y `PublicacionFactory` con datos fake para utilizar en el seeder `UsuarioPublicacionSeeder` para crear 10 usuarios que tentan entre 1 y 5 publicaciones cada uno.

### Práctica: Directorio trabajadores

Desarrolla una app que muestre a los trabajadores de una empresa con sus datos de contacto. Se incorporará un buscador que filtrará por varios de los campos y se darán opciones de ordenación.

A continuación se detallan los requisitos. Deberás hacer las migraciones correspondientes, rutas, controlador, vistas... que necesites para su implementación.

#### Requisitos

1.	Modelo Trabajador con los siguientes campos:

- *nombre* (string): Nombre del trabajador.
- *apellidos* (string): Apellidos del trabajador.
- *telefono* (string): Teléfono profesional.
- *email* (string): Correo electrónico.
- *foto* (string): Nombre de la imagen del trabajador.
- *departamento* (string): Departamento al que pertenece (Ejemplo: Copmras, Ventas, RRHH, I+D...).
- *cargos* (array): Array de strings con los cargos del empleado (Ejemplo: Jefe departamento, coordinador área, director...).
- *fecha_nacimiento* (date): Fecha nacimiento.
- *sustituto* (boolean): Indica si está sustituyendo a otro trabajador.
- *mayor55* (boolean): Indica si es mayor de 55 años.

2.	Funciones CRUD:

- Crear nuevos trabajadores.
- Editar y actualizar trabajadores existentes.
- Eliminar trabajadores.
- Listar todos los trabajadores con opciones de búsqueda filtrado.

3.	Vistas con Blade:

- Listado de trabajadores, con búsqueda y filtros por nombre, apellidos, departamento, cargos y si son o no sustitutos o mayortes de 55 años.
- Formulario para añadir trabajadores.
- Formulario para editar trabajadores.
- Vista de detalle de un trabajador con toda su información.

4.	Extras opcionales:

- Ordenar los trabajadores por fecha de nacimiento (más recientes primero).
- Destacar el trabajador que cumpla años en el día de hoy poniéndole por ejemplo un icono de churros al lado del nombre para la invitación... 😜
  
<!-- 
### Práctica guiada: Fútbol femenino

-->

<!-- 
### Práctica: FernanChollo 

901. Crear el proyecto FernanChollo:

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
  - Un footer con vuestro nombre y algún dato copyright del tipo `©FernanChollo 2025` donde el año debe ser calculado a través de la fecha del servidor.

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

-->