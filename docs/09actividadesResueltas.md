# Laravel Avanzado: Actividades resueltas

A continuación, vas a realizar una serie de ejercicios sencillos sobre cada uno de los apartados vistos en el tema. Puedes crear un proyecto nuevo o reutilizar uno existente.

### Eloquent: Relaciones

En este apartado vas a crear diferentes relaciones entre modelos.

901. **Relación 1 a 1**. 

- Crea los modelos `Usuario` y `Perfil`. Cada usuario tiene un perfil, y cada perfil pertenece a un único usuario.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `usuarios`: campos `id`, `nombre`, `email`.
  - `perfiles`: campos `id`, `usuario_id`, `telefono`, `direccion`.
- Define la relación en los modelos.

??? info "Solución"

    ```php
    // Modelo Usuario
    use Illuminate\Database\Eloquent\Model;

    class Usuario extends Model{
        public function perfil(){
            return $this->hasOne(Perfil::class);
        }
    }

    // Modelo Perfil
    use Illuminate\Database\Eloquent\Model;

    class Perfil extends Model{
        public function usuario(){
            return $this->belongsTo(Usuario::class);
        }
    }
    ```

- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta los datos de un usuario y muestra su perfil. Para ello, crea una ruta `usuario/{id}` que redirija a la función `show` del controlador y llame a la vista `usuario.show.blade.php` para los datos del usuario con su perfil.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $usuario = Usuario::find($id);
        $perfil = $usuario->perfil; // También se podría recuperar el perfil en la vista
        return view('usuario.show', compact('usuario', 'perfil'));
    }
    ```

902. **Relación 1 a Muchos**. 

- Crea los modelos `Categoria` y `Producto`. Cada categoría tiene muchos productos, pero un producto sólo perteneca una determinada categoría.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `categorias`: campos `id`, `nombre`.
  - `productos`: campos `id`, `nombre`, `precio`, `categoria_id`.
- Define la relación en los modelos.

??? info "Solución"

    ```php
    // Modelo Categoria
    use Illuminate\Database\Eloquent\Model;

    class Categoria extends Model{
        public function productos(){
            return $this->hasMany(Producto::class);
        }
    }

    // Modelo Prodcuto
    use Illuminate\Database\Eloquent\Model;

    class Producto extends Model{
        public function categoria(){
            return $this->belongsTo(Categoria::class);
        }
    }
    ```

- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta el nombre de una categoría mostrando sus productos. Para ello, crea una ruta `categoria/{id}` que redirija a la función `show` del controlador y llame a la vista `categoria.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $categoria = Categoria::find($id);
        $productos = $categoria->productos; // También se podría hacer esto en la vista
        return view('categoria.show', compact('categoria', 'productos'));
    }
    ```

- Mediante Eloquent agrega un nuevo producto a la categoría con id pasado por parámetro. Para ello, crea una ruta `categoria/{id}/addproduct/{nombre}` que redirija a la función `addProduct` del controlador y redirija a la ruta `show` anterior que llama a la vista `categoria.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function addProduct(string $id, string $nombre){
        $categoria = Categoria::find($id);
        $categoria->productos()->create(['nombre' => $nombre]);
        
        return redirect()->route('categoria.show', ['id' => $id]);
    }
    ```

903. **Relación Muchos a Muchos**. 

- Crea los modelos `Estudiante` y `Asignatura`. Cada estudiante puede estár matriculado en muchas asignaturas y una asignatura la cursan muchos estudiantes.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `estudiantes`: campos `id`, `nombre`.
  - `asignaturas`: campos `id`, `nombre`.
  - `asignatura_estudiante` (tabla pivote): `estudiante_id`, `asignatura_id`.
- Define la relación en los modelos.

??? info "Solución"

    ```php
    // Modelo Estudiante
    use Illuminate\Database\Eloquent\Model;

    class Estudiante extends Model{
        public function asignaturas(){
            return $this->belongsToMany(Asignatura::class);
        }
    }

    // Modelo Asignatura
    use Illuminate\Database\Eloquent\Model;

    class Asignatura extends Model{
        public function estudiantes(){
            return $this->belongsToMany(Estudiante::class);
        }
    }
    ```

- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta todas las asignaturas de un estudiante por su id. Para ello, crea una ruta `estudiante/{id}` que redirija a la función `show` del controlador y llame a la vista `estudiante.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $estudiante = Estudiante::find($id);
        $asignaturas = $estudiante->asignaturas; // También se podría hacer esto en la vista
        return view('estudiante.show', compact('estudiante', 'asignaturas'));
    }
    ```

- Consulta ahora todos los estudiantes que cursen una asignatura por su id. Para ello, crea una ruta `asignatura/{id}` que redirija a la función `show` del controlador y llame a la vista `asignatura.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $asignatura = Asignatura::find($id);
        $estudiantes = $asignatura->estudiantes; // También se podría hacer esto en la vista
        return view('asignatura.show', compact('asignatura', 'estudiantes'));
    }
    ```

- Mediante Eloquent matricula a un estudiante en un curso determinado. Para ello, crea una ruta `estudiante/{id}/matricula/{idCurso}` que redirija a la función `matricula` del controlador y redirija a la ruta `show` de estudiante que llama a la vista `estudiante.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function matricula(string $id, string $idCurso){
        $estudiante = Estudiante::find($id);
        $estudiante->cursos()->attach($idCurso);
        
        return redirect()->route('estudiante.show', ['id' => $id]);
    }
    ```

- Mediante Eloquent desmatricula a un estudiante en un curso determinado. Para ello, crea una ruta `estudiante/{id}/desmatricula/{idCurso}` que redirija a la función `desmatricula` del controlador y redirija a la ruta `show` de estudiante que llama a la vista `estudiante.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function desmatricula(string $id, string $idCurso){
        $estudiante = Estudiante::find($id);
        $estudiante->cursos()->detach($idCurso);
        
        return redirect()->route('estudiante.show', ['id' => $id]);
    }
    ```

904. **Ejercicio completo**: CRUD con varias relaciones y formularios.

- Implementa un sistema de gestión de posts con sus respectivos comentarios mediante los modelos `Autor`, `Post` y `Comentario`.
- Piensa bien las relaciones a utilizar. Un autor puede escribir muchos posts y comentarios. Un post puede tener muchos comentarios. Un comentario sólo pertenece a un post y está escrito por un autor.
- De un *autor* interesa saber su imagen, nombre y email.
- De un *post* interesa saber su título, fecha y descripción.
- De un *comentario* interesa saber su texto y fecha.
- Crea los CRUDs necesarios para cada modelo con sus rutas específicas, controladores, vistas (con formularios)...

### Mutadores y accesores

910. **Formatear nombres y convertir números**: En el modelo `Producto` del ejercicio anterior, crea:

- Un mutador que almacene el nombre en minúsculas y un accesor que los devuelva con la primera letra en mayúscula.
- Un mutador que almacene el precio convertido a céntimos y un accesor que lo devuelva de nuevo en euros.

??? info "Solución"

    ```php
    namespace App\Models;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Casts\Attribute;

    class Producto extends Model{
        protected $fillable = ['nombre', 'precio'];

        protected function nombre(): Attribute{
            return Attribute::make(
                set: fn ($value) => strtolower($value), // Guardar en minúsculas
                get: fn ($value) => ucfirst($value) // Devolver con la 1ª en mayúsculas
            );
        }

        protected function precio(): Attribute{
            return Attribute::make(
                set: fn ($value) => $value * 100, // Guardar en céntimos
                get: fn ($value) => $value / 100  // Devolver en euros
            );
        }
    }
    ```

911. **Slug automático**: Un slug es una versión formateada de un texto, generalmente usada en URLs por gestores de contenidos. Un slug se crea eliminando caracteres especiales, convirtiendo espacios en guiones y pasando a minúsculas el texto. Por ejemplo: "Hola Mundo Laravel" tendría de slug "hola-mundo-laravel".

En el modelo `Post` del ejercicio anterior:

- Crea la migración correspondiente para añadir el campo `slug` de tipo string.
- Investiga cómo usar `Str::slug` para generar slugs.
- Crea un mutador que convierta el título a slug y lo almacene en el campo `slug`.

??? info "Solución"

    ```php
    <?php
    namespace App\Models;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Support\Str;

    class Post extends Model{
        protected $fillable = ['titulo', 'fecha', 'descripcion', 'slug'];

        protected function titulo(): Attribute{
            return Attribute::make(
                set: function ($value) {
                    return [
                        'titulo' => $value,
                        'slug' => Str::slug($value)
                    ];
                }
            );
        }
    }
    ```

912.  **Formatear fechas de creación**: En el modelo `Estudiante` del ejercicio anterior:

- Investiga cómo usar la biblioteca `Carbon` para trabajar con fechas incluida en Laravel.
- Crea un accesor que formatee la fecha de creación (created_at) en formato "d/m/Y - H:i".

??? info "Solución"

    ```php
    <?php
    namespace App\Models;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Carbon\Carbon;

    class Estudiante extends Model{
        protected $fillable = ['nombre'];

        protected function createdAt(): Attribute{
            return Attribute::make(
                get: fn ($value) => Carbon::parse($value)->format('d/m/Y - H:i')
            );
        }
    }

    // Uso en controlador o vista
    echo $estudiante->created_at; // 04/02/2025 - 15:30
    ```

### Seeders y factories

920. **Seeder básico**: Crea un nuevo modelo `Usuario` con campos `nombre`, `email` y `password` (en su migración) y crea un seeder `UsuarioSeeder` que inserte 3 usuarios de prueba en la base de datos.

??? info "Solución"

    Ejecuta: `php artisan make:seeder UsuarioSeeder`

    ```php
    <?php
    use App\Models\Usuario;
    use Illuminate\Database\Seeder;

    class UsuarioSeeder extends Seeder
    {
        public function run()
        {
            Usuario::create([
                'nombre' => 'Juan Pérez',
                'email' => 'juan@example.com',
                'password' => bcrypt('password123'),
            ]);

            Usuario::create([
                'nombre' => 'Ana Gómez',
                'email' => 'ana@example.com',
                'password' => bcrypt('password123'),
            ]);
            
            Usuario::create([
                'nombre' => 'Adrián Lara',
                'email' => 'adrian@example.com',
                'password' => bcrypt('password123'),
            ]);
        }
    }
    ```

    Añadir al `DatabaseSeeder`:

    ```php
    <?php
    class DatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->call([
                UsuarioSeeder::class,
                // ...
            ]);
        }
    }
    ```

    Ejecuta: `php artisan db:seed --class=UsuarioSeeder`

921. **Factoría con seeder**: Crea una factoría `UsuarioFactory` con datos fake y en el seeder `UsuarioSeeder` crea 10 usuarios mediante la factoría.

??? info "Solución"

    Ejecuta: `php artisan make:factory UsuarioFactory -m Usuario`

    ```php
    <?php
    namespace Database\Factories;
    use App\Models\Usuario;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class UsuarioFactory extends Factory{
        protected $model = Usuario::class; // Se podría obviar 

        public function definition(): array{
            return [
                'nombre' => $faker->name,
                'email' => $faker->unique()->safeEmail,
                'password' => bcrypt('password123'),
            ];
        }
    }
    ```

    En el modelo `Usuario` añadir el trait `use HasFactory` para asociarlos:

    ```php
    <?php
    class Usuario extends Model{
        use HasFactory;
        // ...
    }
    ```

    En el seeder `UsuarioSeeder` creado en el ejercicio anterior, añade:

    ```php
    <?php
    use App\Models\Usuario;
    use Illuminate\Database\Seeder;

    class UsuarioSeeder extends Seeder
    {
        public function run()
        {
            // ...
            User::factory(10)->create();
        }
    }
    ```

    Ejecuta: `php artisan db:seed --class=UsuarioSeeder`

922. **Seeders con modelos relacionados**: Crea el modelo `Publicacion` con los campos `titulo`, `contenido` y `usuario_id` (en su migración) y modifica los modelos para que un usuario se relacione con muchas publicaciones. Crea las factorías `UsuarioFactory` (ya la tienes) y `PublicacionFactory` con datos fake para utilizar en el seeder `UsuarioPublicacionSeeder` para crear 10 usuarios que tentan entre 1 y 5 publicaciones cada uno.

??? info "Solución"

    Ejecuta: `php artisan make:factory PublicacionFactory -m Publicacion`

    ```php
    <?php
    namespace Database\Factories;
    use App\Models\Publicacion;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class PublicacionFactory extends Factory{
        protected $model = Publicacion::class; // Se podría obviar 

        public function definition(): array{
            return [
                'titulo' => $faker->sentence,
                'contenido' => $faker->paragraph,
                'user_id' => 1, // Esto será modificado en el Seeder
            ];
        }
    }
    ```

    En el modelo `Publicacion` añadir el trait `use HasFactory` para asociarlos:

    ```php
    <?php
    class Usuario extends Model{
        use HasFactory;
        // ...
    }
    ```

    Ejecuta: `php artisan make:seeder UsuarioPublicacionSeeder`

    ```php
    <?php
    use App\Models\Usuario;
    use App\Models\Publicacion;
    use Illuminate\Database\Seeder;

    class UsuarioPublicacionSeeder extends Seeder
    {
        public function run()
        {
            // Crear 10 usuarios
            User::factory(10)->create()->each(function ($user) {
                // Crear entre 1 y 5 posts para cada usuario
                $user->posts()->createMany(Post::factory(rand(1, 5))->make()->toArray());
            });
        }
    }
    ```

    Añadir al `DatabaseSeeder`:

    ```php
    <?php
    class DatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->call([
                UsuarioPublicacionSeeder::class,
                // ...
            ]);
        }
    }
    ```

    Ejecuta: `php artisan db:seed --class=UsuarioPublicacionSeeder`