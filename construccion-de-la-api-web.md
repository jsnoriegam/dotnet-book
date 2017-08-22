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

En el ejemplo podemos ver la ruta: **"api/\[controller\]"**, esta ruta es especial porque referencia el nombre del controlador, es decir, para acceder a este controlador debemos ir a http://&lt;dirección-del-servicio&gt;/api/values.  Para que esto funcione el nombre de la clase debe terminar con Controller.

El proyecto esta configurado para serializar los arreglos/objetos retornados por los métodos del controlador a JSON y para deserializar el cuerpo de las peticiones JSON al tipo correspondiente.

### 3.2.1 IActionResult

En el ejemplo anterior podemos ver como los métodos Get retornan objetos o arreglos.  Estos objetos como ya se menció serán transformados a JSON, pero ¿qué pasa si necesitamos mas control sobre las respuestas? Por ejemplo, ¿como hacemos para 

## 3.3 Entity Framework

```xml
<ItemGroup>
  .
  .
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.0.0"/>
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0"/>
</ItemGroup>
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

Para poder trabajar con Entity Framework para el manejo de nuestras entidades y base de datos es necesario crear una clase que extienda de DbContext.

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

Cada DbSet referenciará una tabla en la base de datos.

Lo ideal es no asociar el DbContext con ningún conector de base de datos en particular.

El DbContext debe ser registrado en la clase Startup para que pueda ser utilizado por los repositorios.  En este momento se establece que motor de base de datos se utilizará.

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

Para transformar las entidades en una base de datos se utiliza el comando dotnet ef

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

_Para el caso del proyecto de ejemplo vamos a trabajar con los repositorios como con servicios comunes, puesto que son pocos y no hay procesos que involucren varias entidades, pero es recomendable sobre todo para proyectos mediamos y/o grandes, establecer una clara separación entre servicios y repositorios._

### 3.4.2 Repositorios

Los Repositorios son un tipo especial de servicios cuya finalidad es exponer las operaciones básicas de los repositorios de datos \(tablas\).

### 3.4.2 Inyección de dependencias

Por defecto la inyeccion de dependencias en .NET Core se hace en los contructores

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

## 3.5 Todo en acción

El proyecto de ejemplo lo puede encontrar en github:

