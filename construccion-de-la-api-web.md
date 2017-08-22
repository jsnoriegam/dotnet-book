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

En el ejemplo anterior podemos ver como los métodos Get retornan objetos o arreglos.  Estos objetos como ya se mencioó serán transformados a JSON, pero 

## 3.3 Entity Framework

Par una mayor profundización de los temas puede visitar: [http://www.learnentityframeworkcore.com](http://www.learnentityframeworkcore.com)

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

Para poder trabajar con Entity Framework para el manejo de nuestras entidades y base de datos es necesario crear una clase que extienda de DbContext

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

### 3.4.2 Repositorios

Los Repositorios son un tipo especial de servicios cuya finalidad es exponer las operaciones básicas de los repositorios de datos \(tablas\).

### 3.4.2 Inyección de dependencias

Por defecto la inyeccion de dependencias en .NET Core se hace en los contructores

## 3.5 Todo en acción

```csharp
[Route("api/v1/[controller]")]
public class PeliculasController : Controller
{
    IPeliculasService PeliculasService;
    public PeliculasController(IPeliculasService peliculasService)
    {
        PeliculasService = peliculasService;
    }

    /// <summary>
    /// Obtener una película
    /// </summary>
    /// <param name="id">Database Id de la película solicitada</param>
    [HttpGet("{id}")]
    public IActionResult Get([FromRoute] int id)
    {
        PeliculaWrapperView pelicula = PeliculasService.Obtener(id);
        if (pelicula != null)
        {
            return Ok(pelicula);
        }
        else
        {
            return NotFound();
        }
    }

    /// <summary>
    /// Obtener listado de películas
    /// </summary>
    [HttpGet]
    public IActionResult Get()
    {
        List<PeliculaWrapperView> peliculas = PeliculasService.ObtenerListado();
        if (peliculas != null && peliculas.Count > 0)
        {
            return Ok(peliculas);
        }
        else
        {
            return NoContent();
        }
    }

    /// <summary>
    /// Ingresar una película
    /// </summary>
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
            // Utilizo ToDictionary para obtener solo los datos relevantes del ModelState
            // Para usar ToDictionary se requere System.Linq
            return StatusCode(409, ModelState.ToDictionary(
                ma => ma.Key,
                ma => ma.Value.Errors.Select(e => e.ErrorMessage).ToList()
            ));
        }
    }

    /// <summary>
    /// Modificar una película
    /// </summary>
    [HttpPut("{id}")]
    public IActionResult Put([FromRoute] int id, [FromBody] Pelicula pelicula)
    {
        if (ModelState.IsValid)
        {
            PeliculasService.Modificar(id, pelicula);
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

    /// <summary>
    /// Eliminar una película
    /// </summary>
    [HttpDelete("{id}")]
    public IActionResult Delete([FromRoute] int id)
    {
        PeliculasService.Eliminar(id);
        return Ok();
    }
}
```



