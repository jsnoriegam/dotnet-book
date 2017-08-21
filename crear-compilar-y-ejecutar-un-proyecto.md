# Crear, compilar y ejecutar un proyecto

## Crear un proyecto

Para crear un proyecto con .NET Core debemos crear un directorio, y en una consola dentro del mismo ejecutamos el comando:

```
dotnet new <plantilla-de-proyecto>
```

.NET Core ofrece varias plantillas para crear proyectos segun lo que se desee hacer: aplicaciones web, aplicaciones de consola, o librerías.

Ej.

```
dotnet new console
```

```
dotnet new web
```

```
dotnet new webapi
```

Si deseamos ver todas las plantillas disponibles simplemente ejecutamos:

```
dotnet new
```

Puesto que nuestro objetivo es crear una API web generaremos nuestro proyecto con `dotnet new webapi`.  El proyecto se crea con el nombre del directorio, por lo cual es recomendable utilizar un nombre de una sola palabra con mayúscula inicial. Ej. _Películas._

El comando debe crear la estructura de directorios, el archivo &lt;Nombre-del-proyecto&gt;.csproj, Program.cs, Startup.cs, los archivos de configuración y un contralador de prueba.

## Compilar y ejecutar un proyecto

Para compilar nustro proyeco simplemente ejecutamos el comando:

```
dotnet build
```

Si la compilación no presenta ningún error podemos ejecutar nuestro proyecto utilizando:

```
dotnet run
```

Esto aplica tanto para proyectos web como para consolas.

## Watcher

El proceso de detener la aplicacion compilarla y volverla a ejecutar cada vez que se hace un cambio puede resultar tedioso, por lo que es recomendable agregar la siguiente herramienta al .csproj:

```xml
<DotNetCliToolReference Include="Microsoft.DotNet.Watcher.Tools" Version="2.0.0" />
```

Esta herramienta nos permite ejecutar el comando:

```
dotnet watch
```

El cual se encarga de ejecutar el proyecto y se mantiene activo en background y cuando detecta un cambio se encarga de detener, recompilar y volver a ejecutar.

