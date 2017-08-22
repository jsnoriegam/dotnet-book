# Crear, compilar y ejecutar un proyecto

## 2.1 Crear un proyecto

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

El lenguage predeterminado de .NET Core es C\#, pero se puede utilizar VB o F\# aunque no estan disponibles para todas las plantillas

### 2.1.1 .csproj

Como se mencionó anteriormente uno de los archivos generados dentro del proyecto es &lt;Nombre-del-proyecto&gt;.csproj, Ej. Peliculas.csproj.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <Folder Include="wwwroot\" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
  </ItemGroup>

  <ItemGroup>
    <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />
  </ItemGroup>

</Project>
```

Este archivo contiene las configuraciones basicas de construcción del proyecto asi como las referencias a las librerias y herramientas asociadas al mismo.

Cada vez que se realice un cambio a este archivo se debe ejecutar el comando:

```
dotnet restore
```

## 2.2 Compilar y ejecutar un proyecto

Para compilar nustro proyeco simplemente ejecutamos el comando:

```
dotnet build
```

Si la compilación no presenta ningún error podemos ejecutar nuestro proyecto utilizando:

```
dotnet run
```

Esto aplica tanto para proyectos web como para consolas.

## 2.3 Watcher

El proceso de detener la aplicacion compilarla y volverla a ejecutar cada vez que se hace un cambio puede resultar tedioso, por lo que es recomendable agregar la siguiente herramienta al .csproj:

```xml
<DotNetCliToolReference Include="Microsoft.DotNet.Watcher.Tools" Version="2.0.0" />
```

Esta herramienta nos permite ejecutar el comando:

```
dotnet watch
```

El cual se encarga de ejecutar el proyecto y se mantiene activo en background y cuando detecta un cambio se encarga de detener, recompilar y volver a ejecutar.

