# 06 Consommer une `API` protégé par un `JWT token`

## Page `razor`

POur l'instant la page `razor` : `HRManagement` qui accéder aux données de l'`API` reçoit une erreur `401 Unauthorized` car elle n'est pas authentifiée :

<img src="assets/unauthorized-exception-photo.png" alt="unauthorized-exception-photo" style="zoom:33%;" />

On va s'authentifier depuis la méthode `OnGetAsync` de la `page`.

```cs
public async Task OnGetAsync()
{
    var httpClient = factory.CreateClient("MyClientAPI");

    // De ici
    var response = await httpClient.PostAsJsonAsync(
        "auth",
        new Credential { UserName = "admin", Password = "password" }
    );

    response.EnsureSuccessStatusCode();

    var jwtStr = await response.Content.ReadAsStringAsync();
    
    var token = JsonSerializer.Deserialize<JwtToken>(jwtStr);
    
    httpClient.DefaultRequestHeaders.Authorization 
        = new AuthenticationHeaderValue(
            "Bearer", 
            token?.AccessToken
        );
	// à ici
    
    WeatherforcastItems 
        = await httpClient.GetFromJsonAsync<List<WeatherforecastDto>>(
            "/weatherforecast\n"
        ) ?? [];
}
```

On va en fait ajouter le `header` :

```http
Authorization: bearer eyJhbGciOiJodHRwOi8...
```

La première requête `POST` va retourner notre `Access Token` que nous pourrons passer à l'`API` par le `Header` de la requête `GET`.

```cs
httpClient.DefaultRequestHeaders.Authorization 
        = new AuthenticationHeaderValue(
            "Bearer", 
            token?.AccessToken
        );
```

pourrai tout simplement s'écrire :

```cs
httpClient.DefaultRequestHeaders.Add("Authorization", $"Bearer {token?.AccessToken}");
```

Mais il vaut mieux utiliser les classes de `.net` qui s'occupe des problème d'encodage pour nous.



## Gérer la `Deserialization` avec des noms différents

Si le nom des champs du `Json` et le nom des champs des propriétés de l'objet créé par `JsonSerializer.Deserialize` ne correspondent pas, on doit indiquer à `JsonSerializer` ce qu'il doit faire.

On utilise l'attribut `JsonPropertyName` pour cela :

```cs
public class JwtToken
{
    [JsonPropertyName("access_token")]
    public string AccessToken { get; init; } = string.Empty;
    
    [JsonPropertyName("expire_at")]
    public DateTime ExpireAt { get; init; }
    
}
```

Utile si on ne maitrise pas la convention de nomage de l'`Access Token`.













