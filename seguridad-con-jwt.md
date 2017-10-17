# Seguridad con JWT

La seguridad es un aspecto muy importante para una API web y debe ser implementada de tal manera que podemos seguir manteniendo un esquema RESTful.

Para esto vamos a utilizar JWT \([https://jwt.io/introduction/](https://jwt.io/introduction/)\).

Es recomendable \(o casi obligatorio\) configurar el servidor de producción para que utilice una conexión encriptada ssl/tls \(https\) para que el sistema tenga un grado de confianza real en seguridad de la información.  Pero para el desarrollo de la API la utilización de http no genera ningún inconveniente.

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

Básicamente debemos establecer que vamos a validar del token y la llave que vamos a utilizar para encriptarlo, los datos del ejemplo son lo mínimo recomendado.

Los datos de configuración podemos incluirlos en appsettings.json:

```json
"AuthenticationSettings": {
    "Issuer": "Movies API",
    "Audience": "Public",
    "SigningKey": "khCUzyz5DwZK1BIoTN0csKQl4o6xFd8h"
}
```

Tenga en cuenta que la llave NO debe ser de conocimiento público y debe ser mantenida en secreto.

Luego dentro del método **Configure **activamos el filtro JWT agregando:

```csharp
app.UseAuthentication();
```

antes de la llamada a **app.UseMvc\(\)**.

Para establecer que un controlador debe requiere autorización debemos agregar el atributo Authorize al controlador:

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

Tambien podemos agregar el atributo a los métodos para un mayor nivel de personalización:

```csharp
[HttpDelete("{id}")]
[Authorize(Roles = "ADMIN")]
public IActionResult Delete([FromRoute] int id)
{
    PeliculasService.Eliminar(id);
    return Ok();
}
```

En este caso solo dejará efectuar la operación si el usuario tiene el Rol ADMIN. En el siguitente tema se muestra como se registran los roles en un token.

## 5.1 Generar un Token JWT

Ahora es necesario tener la capacidad de generar el token que vamos a utilizar para autenticar las peticiones que realizamos a la API.

La forma mas sencilla es crear un controlador que reciba un usuario y un password y nos retorne dicho token:

```csharp
[Route("api/v1/[controller]")]
public class AuthController : Controller
{
    IAuthService AuthService;

    public AuthController(IAuthService authService)
    {
        AuthService = authService;
    }
    [HttpPost("token")]
    public IActionResult Token([FromBody] UserContext context)
    {
        if (ModelState.IsValid && AuthService.ValidateUser(context.Username, context.Password))
        {
            var now = DateTime.UtcNow;
            var validTime = TimeSpan.FromHours(2);
            var expires = now.Add(validTime);
            var token = AuthService.GenerateAccessToken(now, context.Username, validTime);
            return Ok(new {
                Token = token,
                ExpiresAt = expires
            });
        }
        else
        {
            return StatusCode(401);
        }
    }
}

public class UserContext
{
    [Required]
    public string Username { get; set; }
    [Required]
    public string Password { get; set; }
}
```

La clase UserContext se utiliza para recibir los datos de la petición.

Luego creamos la clase **AuthService **en la cual efectuaremos la validación de las credenciales y generaremos el correspondiente token:

```csharp
public class AuthService : IAuthService
{
    AuthSettings Settings;

    public AuthService(IOptions<AuthSettings> options)
    {
        Settings = options.Value;
    }
    // Utilice su propia lógica de validación de usuarios
    public bool ValidateUser(string username, string password)
    {
        return username.Equals("admin") && password.Equals("admin");
    }
    public string GenerateAccessToken(DateTime now, string username, TimeSpan validtime)
    {
        var expires = now.Add(validtime);
        var claims = new Claim[]
        {
                new Claim(JwtRegisteredClaimNames.Sub, username),
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
                new Claim(
                    JwtRegisteredClaimNames.Iat,
                    new DateTimeOffset(now).ToUniversalTime().ToUnixTimeSeconds().ToString(),
                    ClaimValueTypes.Integer64
                ),
                new Claim(
                    "roles",
                    "ADMIN"
                ),
                new Claim(
                    "roles",
                    "SUPERUSUARIO"
                )
        };
        var signingCredentials = new Microsoft.IdentityModel.Tokens.SigningCredentials(
            new SymmetricSecurityKey(Encoding.ASCII.GetBytes(Settings.SigningKey)),
            SecurityAlgorithms.HmacSha256Signature
        );
        var jwt = new JwtSecurityToken(
            issuer: Settings.Issuer,
            audience: Settings.Audience,
            claims: claims,
            notBefore: now,
            expires: expires,
            signingCredentials: signingCredentials
        );
        var encodedJwt = new JwtSecurityTokenHandler().WriteToken(jwt);
        return encodedJwt;
    }
}

public interface IAuthService {
    bool ValidateUser(string username, string password);
    string GenerateAccessToken(DateTime now, string username, TimeSpan validtime);
}
```

El claim **roles **se utiliza para registrar uno o mas roles asociados al token y es requerido para poder utilizar: **\[Authorize\(Roles = "ADMIN"\)\]**.

> Tenga en cuenta que **roles **no hace parte de los nombres registrados para claims en la especificación de JWT \([https://tools.ietf.org/html/rfc7519\#section-4.1](https://tools.ietf.org/html/rfc7519#section-4.1)\).  Lo cual no implica en ningún momento que sea inválido.

Cabe anotar que generamos los tiempos utilizando UTC para tener un mejor control del tiempo de vida del token.

Para probar si el controlador funciona utilizamos nuestro cliente HTTP preferido:

```
Method: POST
Headers:
    Content-Type: application/json
Body: 
    {
        "username": "admin",
        "password": "admin"
    }
```

Para lo cual se debería obtener una respuesta similar a esta:

**200 Ok**

```json
{
"token": "eyJhbGciOiJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGRzaWctbW9yZSNobWFjLXNoYTI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhZG1pbiIsImp0aSI6IjRhODFkYzI1LWQ5YWItNDk5MS05Y2MyLTE5NTM2MTE0YmY1NCIsImlhdCI6MTUwMzExMzQ1MCwicm9sZXMiOlsiQURNSU4iLCJTVVBFUlVTVUFSSU8iXSwibmJmIjoxNTAzMTEzNDUwLCJleHAiOjE1MDMxMjA2NTAsImlzcyI6Ik1vdmllcyBBUEkiLCJhdWQiOiJQdWJsaWMifQ.qiK88t1w3cYZhqjS9TSnv-o9v3AUnvLpJVVxa9CBAxc",
"expiresAt": "2017-08-19T05:30:50.3300999Z"
}
```

Para utilizar el token solo debemos añadir el header **Authorization **a cada una de nuestras peticiones, con el identificador Bearer seguido del token.

```
Authorization: Bearer eyJhbGciOiJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGRzaWctbW9yZSNobWFjLXNoYTI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhZG1pbiIsImp0aSI6IjRhODFkYzI1LWQ5YWItNDk5MS05Y2MyLTE5NTM2MTE0YmY1NCIsImlhdCI6MTUwMzExMzQ1MCwicm9sZXMiOlsiQURNSU4iLCJTVVBFUlVTVUFSSU8iXSwibmJmIjoxNTAzMTEzNDUwLCJleHAiOjE1MDMxMjA2NTAsImlzcyI6Ik1vdmllcyBBUEkiLCJhdWQiOiJQdWJsaWMifQ.qiK88t1w3cYZhqjS9TSnv-o9v3AUnvLpJVVxa9CBAxc
```



