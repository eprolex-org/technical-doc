# 07 Créer une `Session`

> une **session** est une période délimitée pendant laquelle un appareil informatique est en communication et réalise des opérations au service d'un client - un usager, un logiciel ou un autre appareil.



## Ajout d'une `Session` 

La `Session` est gérée par les `Cookies`, on va stocker le `jwt` dans un `Cookie`.

### Ajout su service : `Sevices.AddSession`

```cs
builder.Services.AddSession(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
    options.Cookie.Name = "MySessionCookie";
    options.IdleTimeout = TimeSpan.FromMinutes(30);
});
```

`options.Cookie.HttpOnly = true` détermine si le `Cookie` est accessible à un script (`js`), `true` signifie qu'il n'est pas accessible.

` options.Cookie.IsEssential = true` le `Cookie` peut être envoyé sans le consentement de l'utilisateur.

`options.IdleTimeout` donne le temps effectif de la `Session`, `idle` voulant dire `inactif`.



### Ajout d'un middleware `UseSession`

```c#
app.UseAuthentication();
app.UseAuthorization();

app.UseSession(); // <- ici

app.MapRazorPages();
```



## Utilisation dans une page

`Weather.cshtml.cs`

```cs
public class Weather(IHttpClientFactory factory) : PageModel
{
    [BindProperty] public List<WeatherDto> WeatherItems { get; set; } = [];

    public async Task OnGetAsync()
    {
        JwtToken token = new();

        var tokenSessionStr = HttpContext.Session.GetString("access_token");

        token = string.IsNullOrEmpty(tokenSessionStr)
            ? await Authenticate()
            : JsonSerializer.Deserialize<JwtToken>(tokenSessionStr) ?? new JwtToken();

        if (token.AccessToken == string.Empty || token.ExpireAt <= DateTime.Now)
            token = await Authenticate();

        var client = factory.CreateClient(MyHttpClient.ClientWebApi);
        
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue(
            "Bearer",
            token.AccessToken);
        
        WeatherItems = await client.GetFromJsonAsync<List<WeatherDto>>("/weatherforecast") ?? [];
    }
```

La `session` est accessible via `HtpContext.Session`.

Si le `token` trouvé dans la `session` est `null` ou vide, on s'identifie et on reçoit un nouveau `token` avec `await Authenticate` sinon on le déserialize.

```cs
private async Task<JwtToken> Authenticate()
    {
     	var client = factory.CreateClient(MyHttpClient.ClientWebApi);

        var response = await client.PostAsJsonAsync(
            "/auth",
            new Credential
            {
                UserName = "admin",
                Password = "password",
                Department = "HR"
            });

        response.EnsureSuccessStatusCode();

        var tokenString = await response.Content.ReadAsStringAsync();

        HttpContext.Session.SetString("token_access", tokenString);

        return JsonSerializer.Deserialize<JwtToken>(tokenString) ?? new JwtToken();
    }
```

