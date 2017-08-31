# Pruebas unitarias

## 5.1 MSTest

Para crear una suite de pruebas para nuesto proyecto debemos crear otro proyecto \(lo ideal es tener ambos proyectos en un mismo superdirectorio\).

```
dotnet new mstest
```

Para nuestro proyecto de Peliculas crearemos un directorio llamado Peliculas.Test.

una vez creado el proyecto es necesario que agreguemos una referencia al proyecto al cual le vamos a aplicar las pruebas, para lo cual podemos utilizar el siguiente comando:

```
 dotnet add reference ..\Peliculas\Peliculas.csproj
```

O podemos agregar manualmente la referencia directamente en el .csproj del proyecto de Test:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>

    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="15.3.0-preview-20170628-02" />
    <PackageReference Include="MSTest.TestAdapter" Version="1.1.18" />
    <PackageReference Include="MSTest.TestFramework" Version="1.1.18" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Peliculas\Peliculas.csproj" />
  </ItemGroup>

</Project>
```

Luego solo debemos crear las clases con los métodos que efectuaran las pruebas correspondientes. Se recomienda crear una clase para cada servicio del proyecto referenciado.

```csharp
[TestClass]
public class UnitTestPeliculasService
{
    PeliculasContext Context;
    IPeliculasService PeliculasService;
    public UnitTestPeliculasService() {
        Context = new PeliculasContext(new DbContextOptionsBuilder<PeliculasContext>().UseInMemoryDatabase(databaseName: "Peliculas").Options);
        PeliculasService = new PeliculasService(Context);
    }
    [TestInitialize]
    public void TestInitialize()
    {
        //Datos iniciales de nuestra base de datos en memoria
        var persona = new Persona() {
            Nombre = "Zack",
            Apellido = "Snyder",
            CodigoIMDB = "nm0811583"
        };
        Context.Personas.Add(persona);
        var pelicula = new Pelicula() {
            Nombre = "Batman v Superman: Dawn of Justice",
            CodigoIMDB = "tt2975590",
            Resumen = "Fearing that the actions of Superman are left unchecked, Batman takes on the Man of Steel, while the world wrestles with what kind of a hero it really needs.",
            Director = persona
        };
        Context.Peliculas.Add(pelicula);
        Context.SaveChanges();
    }
    [TestMethod]
    public void TestMethodAgregar()
    {
        Console.WriteLine("Agregar una película");
        var pelicula = new Pelicula() {
            Nombre = "Wonder Woman",
            CodigoIMDB = "tt0451279"
        };
        PeliculasService.Agregar(pelicula);
        Assert.AreEqual(2, Context.Peliculas.ToList().Count);
    }
    [TestMethod]
    public void TestMethodListar()
    {
        Console.WriteLine("Obtener listado de películas");
        var peliculas = PeliculasService.ObtenerListado();
        Assert.IsInstanceOfType(peliculas, typeof(List<PeliculaWrapperView>));
    }
}
```

Debemos marcar cada clase de pruebas con el atributo **TestClass** y cada método con el atributo **TestMethod**, si necesitamos ejecutar un proceso para cada prueba podemos marcar un método con **TestInitialize**.

Cabe resaltar que configuramos el context con UseInMemoryDatabase para asegurar la predictibilidad y el correcto funcionamiento de las pruebas.

Una vez tengamos definidos todas las pruebas ejecutamos:

```
dotnet test
```

O si tenemos instalado watcher, podemos ejecutar:

```
dotnet watch test
```



