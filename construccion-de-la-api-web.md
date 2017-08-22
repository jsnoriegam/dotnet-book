# Construcción de la API web

Para el desarrollo de este documento crearemos una API web para llevar una base de datos de películas \(Como IMDB pero edición ligera\)

Solo manejaremos 2 entidades: Pelicula y Persona

## Startup.cs

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

## Controladores

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

### IActionResult

En el ejemplo anterior podemos ver como los métodos 

## DbContext

Para poder trabajar con Entity Framework para el manejo de nuestras entidades y base de datos es necesario crear una clase que extienda de DbContext

## Entidades

## Servicios

Crear un servicio implica solo crear una clase con sus correspondientes métodos

### Inyección de dependencias

## Todo en acción



