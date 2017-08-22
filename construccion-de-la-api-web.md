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

## DbContext

## Servicios



