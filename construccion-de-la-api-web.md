# Construcción de la API web

Para el desarrollo de este documento crearemos una API web para llevar una base de datos de películas \(Como IMDB pero edición ligera\)

Solo manejaremos 2 entidades: Pelicula y Persona

## 3.1 Startup.cs

Cada proyecto web debe contener una Clase Startup en la cual se realizarán las configuraciones globales y ser referenciaran los servicios, filtros y/o middlewares que el proyecto necesite.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;

namespace Peliculas
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseMvc();
        }
    }
}
```

Todo servicio debe ser registrado en esta clase para que pueda ser utilizado posteriormente, ya sea en los controladores o en otros servicios.

## 3.2 Controladores

La creación de controladores en .NET Core es muy sencilla, solo se necesita crear una clase que extienda de **Controller **y marcarla con el atributo **Route**

Este es el controlador de ejemplo que se genera al crear el proyecto:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace Peliculas.Controllers
{
    [Route("api/[controller]")]
    public class ValuesController : Controller
    {
        // GET api/values
        [HttpGet]
        public IEnumerable<string> Get()
        {
            return new string[] { "value1", "value2" };
        }

        // GET api/values/5
        [HttpGet("{id}")]
        public string Get(int id)
        {
            return "value";
        }

        // POST api/values
        [HttpPost]
        public void Post([FromBody]string value)
        {
        }

        // PUT api/values/5
        [HttpPut("{id}")]
        public void Put(int id, [FromBody]string value)
        {
        }

        // DELETE api/values/5
        [HttpDelete("{id}")]
        public void Delete(int id)
        {
        }
    }
}
```

En este controlador podemos ver que mediante el atributo Route establecemos la ruta a la cual responderá el controlador, además vemos como cada método esta marcado con atributos que identifican a que verbo HTTP responderán.

En el ejemplo podemos ver la ruta: **"api/\[controller\]"**, esta ruta es especial porque referencia el nombre del controlador, es decir, para acceder a este controlador debemos ir a [http://&lt;dirección-del-servicio&gt;/api/values](http://<dirección-del-servicio>/api/values).  Para que esto funcione el nombre de la clase debe terminar con Controller.

El proyecto esta configurado para serializar los arreglos/objetos retornados por los métodos del controlador a JSON y para deserializar el cuerpo de las peticiones JSON al tipo correspondiente.

### 3.2.1 IActionResult

En el ejemplo anterior podemos ver como los métodos Get retornan objetos o arreglos.  Estos objetos como ya se menció serán transformados a JSON, pero ¿qué pasa si necesitamos mas control sobre las respuestas? Por ejemplo, ¿como hacemos para mandar un error de validación?



## 3.3 Entity Framework

La referencia **Microsoft.AspNetCore.All** del .csproj incluye todas las librerias necesarias para trabajar con Entity Framework Core, pero si queremos activar las herramientas de línea de comandos debemos agregar:

```xml
<ItemGroup>
  .
  .
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.0.0"/>
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0"/>
</ItemGroup>
```

Estas referencias nos permiten utilizar las herramientas de línea de comandos **dotnet ef**.

Además se incluye soporte para MS Sql Server, Sqlite e InMemory database.

Si queremos agregar soporte para MySQL podemos agregar la referencia:

```xml
<PackageReference Include="Pomelo.EntityFrameworkCore.MySql" Version="2.0.0-rtm-10056"/>
```

Para mayor profundización del tema puede visitar: [http://www.learnentityframeworkcore.com](http://www.learnentityframeworkcore.com)

### 3.3.2 Entidades

Las entidades son las clases que referenciaran el modelo de datos de nuestro proyecto.  Deben tener un atributo llamado Id o &lt;nombre-de-la-entidad&gt;Id o marcado con un atributo **Key**.

```csharp
public class Pelicula
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }
    [Required]
    [MaxLength(256)]
    public string Nombre { get; set; }
    [Required]
    [MaxLength(32)]
    public string CodigoIMDB { get; set; }
    public string Resumen { get; set; }
    [ForeignKey("DirectorId")]
    public Persona Director { get; set; }
    public int? DirectorId { get; set; }
}
```

```csharp
public class Persona
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }
    [Required]
    [MaxLength(256)]
    public string Nombre { get; set; }
    [Required]
    [MaxLength(256)]
    public string Apellido { get; set; }
    [NotMapped]
    public string NombreCompleto => $"{Nombre} {Apellido}";
    [MaxLength(32)]
    public string CodigoIMDB { get; set; }
    [InverseProperty("Director")]
    public ICollection<Pelicula> DirectorDe { get; set; }
}
```

Tambien podemos marcar ciertas propiedades con atributos para definir llaves foraneas \(many-to-one\) o para crear restricciones

### 3.3.1 DbContext

Para poder trabajar con Entity Framework para el manejo de nuestras entidades y conexiones a bases de datos es necesario crear una clase que extienda de DbContext.

Esta clase debe referenciar todas las entidades mediante el uso de **DbSet**

```csharp
public class PeliculasContext : DbContext
{
    //Los BbSet representarán las tablas referenciadas por cada entidad
    public DbSet<Pelicula> Peliculas { get; set; }
    public DbSet<Persona> Personas { get; set; }
    public PeliculasContext(DbContextOptions<PeliculasContext> options) : base(options)
    {
        //Database.EnsureCreated();//Crea la BD y las tablas según el esquema actual (No actualiza!!)
        //Database.Migrate(); //Ejecuta cualquier migración que este pendiente
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        //Sobreescriba este método para configuraciones adicionales de las entidades
        modelBuilder.Entity<Pelicula>().HasIndex(p => p.CodigoIMDB).IsUnique();
        base.OnModelCreating(modelBuilder);
    }
}
```

Cada DbSet representará una tabla en la base de datos.

Lo ideal es no asociar el DbContext con ningún conector de base de datos en particular, en especial si se va a utilizar el mismo DbContext para pruebas unitarias.

El DbContext debe ser registrado en la clase Startup para que pueda ser utilizado por los repositorios.  En este momento se establece que motor de base de datos se utilizará.

Las cadenas de conexión podemos registrarla en appsettings.json:

```json
"ConnectionStrings": {
    "MySql": "Server=localhost; UserId=root; Password=; Database=peliculas"
}
```

Y luego dentro de **ConfigureServices **agregamos lo siguiente:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var connectionString = Configuration.GetConnectionString("MySql");
    // Add framework services.
    services.AddDbContext<PeliculasContext>(options =>
        options.UseMySql(connectionString)
    );
    .
    .
    .
}
```

### 3.1.2 Dotnet ef

Para transformar las entidades en una base de datos se utiliza el comando dotnet ef, lo mas común es hacer migraciones para cada cambio que se haga a las entidades:

```
dotnet ef migrations add <Nombre-de-la-migracion>
```

Con este comando se creará el directorio Migrations en nuestro proyecto y las clases que permitiran la generación automática del esquema en la base de datos.

Para ejecutar las migraciones se utiliza:

```
dotnet ef database update
```

### 3.4 Servicios

Para crear un servicio solo se debe crear una una interface y por lo menos una implementación con sus correspondientes métodos.

Para poder inyectar los servicios es necesario registrarlos en Startup:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    .
    .
    .
    services.AddScoped<IPeliculasService, PeliculasService>();
    services.AddScoped<IPersonasService, PersonasService>();
    services.AddMvc();
}
```

> _Para el caso del proyecto de ejemplo vamos a trabajar con los repositorios como si se tratara de servicios, puesto que son pocos y no hay procesos que involucren varias entidades, pero es recomendable sobre todo para proyectos mediamos y/o grandes, establecer una clara separación entre servicios y repositorios._

### 3.4.2 Repositorios

Los Repositorios son un tipo especial de servicios cuya finalidad es exponer las operaciones básicas de los repositorios de datos \(tablas\).

Por lo general los repositorios solo tienen funciones que implican operaciones analogas a un listado: obtenerUno, obtenerTodos, agregar, reemplazar y eliminar aunque pueden tener otras mientras sean sobre la misma entidad.

### 3.4.2 Inyección de dependencias

Por defecto la inyeccion de dependencias en .NET Core se hace en los contructores, solo es necesario definir el servicio a inyectar como parámetro del constructor y el framework se encargará del resto.

```csharp
[Route("api/v1/[controller]")]
public class PeliculasController : Controller
{
    IPeliculasService PeliculasService;
    public PeliculasController(IPeliculasService peliculasService)
    {
        PeliculasService = peliculasService;
    }
    .
    .
    .
}
```

La inyección de IPeliculasService se hace automáticamente si la implementación esta registrada en Startup.

## 3.5 CORS

Si nuestra API va a ser utilizada con clientes javascript es necesario registrar el filtro CORS para evitar problemas en caso de que sea necesario que el cliente se encuentre en un dominio diferente.

Una configuración básica de CORS la podemos hcer de la siguiente manera:

En el método ConfigureServices de Startup agregamos lo siguiente antes de services.AddMvc\(\):

```csharp
services.AddCors(options =>
{
    options.AddPolicy("CorsPolicy",
        builder => builder.AllowAnyOrigin()
        .AllowAnyMethod()
        .AllowAnyHeader()
        .AllowCredentials());
});
```

Y en el método Configure antes de app.UseMvc\(\):

```csharp
app.UseCors("CorsPolicy");
```

Mas información en: [https://docs.microsoft.com/en-us/aspnet/core/security/cors](https://docs.microsoft.com/en-us/aspnet/core/security/cors)

## 3.6 Validación

El proceso de validación de las entidades se hace automáticamente al momento de construir el objeto de tipo Película que se inyectará en el parámetro del método **Post** según los atributos establecidos en la entidad\(Ej. Required\), pero se deja a opción al desarrollador que pasos seguir en caso de que no se cumpla alguna de las resticciones.

Para verificar si se pasó o no la validación debemos verificar el estado de ModelState:

```csharp
[HttpPost]
public IActionResult Post([FromBody] Pelicula pelicula)
{
    if (ModelState.IsValid)
    {
        PeliculasService.Agregar(pelicula);
        return Ok();
    }
    else
    {
        return StatusCode(409, ModelState.ToDictionary(
            ma => ma.Key,
            ma => ma.Value.Errors.Select(e => e.ErrorMessage).ToList()
        ));
    }
}
```

## 3.7 Presentación de la respuesta

A veces es necesario cambiar la forma en que presentamos los datos, ya sea para mostrar u ocultar información, o para evitar problemas como referencias circulares.  En estos casos se puede implementar una solucón creando un DTO utilizando una herramienta como Automapper, o una solución mas sencilla es encapsular el objeto original en un wrapper \(estilo delegado\) y crear getters solo a los datos que se deseen mostrar.

```csharp
public class PeliculaWrapperView
{
    protected Pelicula Pelicula;
    public PeliculaWrapperView(Pelicula pelicula)
    {
        Pelicula = pelicula;
    }

    public int Id { get => Pelicula.Id; }
    public string Nombre { get => Pelicula.Nombre; }
    public string CodigoIMDB { get => Pelicula.CodigoIMDB; }
    public string Resumen { get => Pelicula.Resumen; }
    public PersonaWrapperPeliculaView Director { get => new PersonaWrapperPeliculaView(Pelicula.Director); }
}

public class PersonaWrapperPeliculaView {
    Persona Persona;
    public PersonaWrapperPeliculaView(Persona persona)
    {
        Persona = persona;
    }
    public int Id { get => Persona.Id; }
    public string NombreCompleto { get => Persona.NombreCompleto; }
    public string CodigoIMDB { get => Persona.CodigoIMDB; }
}
```

Luego solo es cuestión de modificar los servicios necesarios:

```csharp
public PeliculaWrapperView Obtener(int id) {
    return new PeliculaWrapperView(Context.Peliculas.Include(p => p.Director).First(p => p.Id == id));
}
public List<PeliculaWrapperView> ObtenerListado()
{
    //Es necesario referenciar System.Linq
    //Include p.Director genera internamete un JOIN en la consulta
    return Context.Peliculas.AsNoTracking().Include(p => p.Director).Select(p => new PeliculaWrapperView(p)).ToList();
}
```

Es necesario desactivar la generación de proxies con .AsNoTracking\(\) para poder utilizar el wrapper sobre una consulta con Linq, ya que estamos reemplazando la entidad real por otro objeto.

## 3.8 Todo en acción

El proyecto de ejemplo lo puede encontrar en github: [https://github.com/jsnoriegam/dotnetmovies2.git](https://github.com/jsnoriegam/dotnetmovies2.git)

Podemos utilizar extensiones como Restlet o Postman de chorme para realizar pruebas a la API:

Petición:

```
URL: <direccion-api>/api/v1/peliculas
Method: POST
Headers:
    Content-Type: application/json
Body:
    { 
        "nombre" : "Batman v Superman: Dawn of Justice",
        "codigoIMDB" : "tt2975590",
        "resumen" : "Fearing that the actions of Superman are left unchecked, Batman takes on the Man of Steel, while the world wrestles with what kind of a hero it really needs."
    }
```

Respuesta:

**200 Ok**

Petición:

```
URL: <direccion-api>/api/v1/peliculas
Method: GET
```

Respuesta:

**200 Ok**

```
[
    {
        "id":1,
        "nombre":"Batman v Superman: Dawn of Justice",
        "codigoIMDB":"tt2975590",
        "resumen":"Fearing that the actions of Superman are left unchecked, Batman takes on the Man of Steel, while the world wrestles with what kind of a hero it really needs.",
        "director": null
    }
]
```



