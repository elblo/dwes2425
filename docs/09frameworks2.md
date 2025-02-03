# Uso avanzado de Frameworks

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








## 9.X Eloquent: Relaciones

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
      return $this -> belongsTo(Usuario::class);
  }
}
```

Al llamar el método de `usuario`, Eloquent intentará encontrar un modelo Usuario que tenga un `id` que coincida con la columna de `usuario_id` en el modelo de `telefono`.

Eloquent determina el nombre de la clave externa examinando el nombre del método de relación y agregando el sufijo `_id` al nombre del método. Entonces, asume que el modelo `Telefono` tiene una columna `usuario_id`. Sin embargo, si no se llama de esa manera, puedes pasarle como argumento el nombre de la clave.

```php
<?php
public function usuario()
{
    return $this -> belongsTo(Usuario::class, 'clave_ajena');
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

## 9.2 Mutadores y accesores

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

## 9.3 Seeders y factorías




---






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
