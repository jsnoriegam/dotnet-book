# Seguridad con JWT

La seguridad es un aspecto muy importante para una API web y debe ser implementada de tal manera que podemos seguir manteniedo un esquema RESTful.

Para esto vamos a utilizar JWT \([https://jwt.io/introduction/](https://jwt.io/introduction/)\).

## 5.1 Configuración de Startup

Lo primero que debemos hacer es registrar el servicio en el método **ConfigureServices**:

```csharp
services.AddAuthorization(options =>
{
    options.DefaultPolicy = new AuthorizationPolicyBuilder(JwtBearerDefaults.AuthenticationScheme)
        .RequireAuthenticatedUser()
        .Build();
});

var issuer = Configuration["AuthenticationSettings:Issuer"];
var audience = Configuration["AuthenticationSettings:Audience"];
var signingKey = Configuration["AuthenticationSettings:SigningKey"];

services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(o =>
{
    o.Audience = audience;
    o.TokenValidationParameters = new TokenValidationParameters()
    {
        ValidateIssuer = true,
        ValidIssuer = issuer,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(signingKey))
    };
});
```

Los datos de configuración deben estar en appsettings.json:

```json
"AuthenticationSettings": {
    "Issuer": "Movies API",
    "Audience": "Public",
    "SigningKey": "khCUzyz5DwZK1BIoTN0csKQl4o6xFd8h"
}
```

Tenga en cuenta que la llave NO debe ser de conocimiento público y debe ser mantenida en secreto.

Luego dentro del método Configure activamos el filtro agregando:

```csharp
app.UseAuthentication();
```

antes de app.UseMvc\(\).

Para establecer que un controlador debe ser autenticado debemos agregar el atributo Authorize al controlador:

```csharp
[Authorize]
[Route("api/v1/[controller]")]
public class PeliculasController : Controller
{
    .
    .
    .
}
```

Tambien podemos 

