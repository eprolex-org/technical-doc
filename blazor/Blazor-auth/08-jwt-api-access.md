# Passer un `JWT` pour accéder à l'`API`

## Depuis un composant : `AuthenticationStateProvider`

### Dans un service appelé dans un composant

```cs
public class ServiceForComponent(IHttpClientFactory httpClientFactory, AuthenticationStateProvider authenticationState)
{
    private readonly IHttpClientFactory _httpClientFactory = httpClientFactory;
    private readonly AuthenticationStateProvider _authenticationState = authenticationState;

    protected async  Task<HttpClient> CommunicateToAPIAsync()
    {

        var authState = await _authenticationState.GetAuthenticationStateAsync();

        var user = authState.User;
        
        var httpClient = _httpClientFactory.CreateClient(API_NAME);

        if (user.Identity is null || !user.Identity.IsAuthenticated)
        {
            Console.WriteLine("user is null");
            
            return httpClient;
        }
        
        var jwt = JwtTokenService.GetToken(user);
        
        httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", jwt);
        
        await httpClient.GetAsync(...);
    
```

On récupère l'utilisateur `authState.User`, on créé un `JWT` avec les `Claims` du `User`, puis on les passes par le `Header` `Authorization : Bearer <jwt>`.

`JwtTokenService`

```cs
public static class JwtTokenService
{
    public static string GetToken(ClaimsPrincipal user)
    {

        var utilisateurId = user.FindFirstValue(ClaimTypes.NameIdentifier);

        List<Claim> claims = [];
        
        if(utilisateurId is not null) claims.Add(new Claim(JwtRegisteredClaimNames.Sub, utilisateurId));
            
        claims.AddRange(user.FindAll(ClaimTypes.Role).Select(role => new Claim("role", role.Value)));

        var key = new SymmetricSecurityKey("security key"u8.ToArray());

        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: "blazor",
            audience: "api",
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: creds);

        var jwt = new JwtSecurityTokenHandler().WriteToken(token);

        return jwt;
    }
}
```

Si le service est appelé en dehors du scope d'un composant on reçoit une `exception` :
```
Une exception non gérée s'est produite lors de l'exécution de la requête.
System.InvalidOperationException : n'appelez pas GetAuthenticationStateAsync en dehors de la portée DI pour un composant Razor. En général, cela signifie que vous ne pouvez l'appeler qu'au sein d'un composant Razor ou d'un autre service DI résolu pour un composant Razor.
```



### Directement depuis un `Razor Component` : `Task<AuthenticationState>`

```cs
@inject DemandeAvisState DemandeAvisState
@inject IHttpClientFactory HttpClientFactory


@foreach (var delegue in delegues)
{
    <EpDelegue @key="delegue" Delegue="delegue"/>
}


@code {
    [CascadingParameter] 
    public required Task<AuthenticationState> AuthenticationTask { get; set; }

    List<Delegue> delegues = [];

    protected override async Task OnInitializedAsync()
    {
        await LoadItemsAsync();
    }

    async Task LoadItemsAsync(bool isAsyncChanged = false)
    {
        var authState = await AuthenticationTask;
        var user = authState.User;
        
        var jwt = JwtTokenService.GetToken(user);
        
        var httpClient = HttpClientFactory.CreateClient(EProLexAPI.ClientName);
        httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", jwt);
        
        delegues = await httpClient.GetFromJsonAsync<IReadOnlyList<Delegue>>(
            $"demande-avis/{DemandeAvisState.DemandeAvis.Id}/delegue"
        );
    }
}
```

