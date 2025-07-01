# 07 Créer une `Policy`

## Dans `AddAuthorization`

On peut ajouter des `policy` directement dans les options du service `AddAuthorization` :

```cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("MichelineAdminPolicy", policy =>
    {
        policy.RequireRole("Admin");
        policy.RequireUserName("Micheline");
    });
}
```



## Création d'une `policy` dans une classe séparée

`LetterAMandatoryRequirement.cs`

```cs
public class LetterAMandatoryRequirement : IAuthorizationRequirement { }
```

`LetterAMandatoryRequirementHandler.cs`

```cs
public class LetterAMandatoryRequirementHandler() : AuthorizationHandler<LetterAMandatoryRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, 
        LetterAMandatoryRequirement requirement
    )
    {
        var name = context.User.Identity?.Name ?? string.Empty;
        Console.WriteLine("Auth handler appelé");

        if (name.Contains('a'))
        {
            context.Succeed(requirement);
        }
        else if (!context.User.Identity?.IsAuthenticated ?? true)
        {
            context.Fail(
                new AuthorizationFailureReason(this, "vous n'êtes pas authentifié")
            );
            
        }
        else
        {
            context.Fail(
                new AuthorizationFailureReason(this, "votre nom ne contient pas de 'a'")
            );
        }

        return Task.CompletedTask;
    }
}
```

`Fail` et `Succed` vont nous aider à créer la logique de notre `handler`.



## Enregistrement dans `Program.cs`

```cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy(
        "LetterPolicy",
        policy => policy.Requirements.Add(new LetterAMandatoryRequirement())
    );  
});

builder.Services.AddScoped<IAuthorizationHandler, LetterAMandatoryRequirementHandler>();
```



## Utilisation de la `policy`

Pour la page entière :

```ruby
@page "/counter"

@attribute [Authorize(Policy = "LetterPolicy")]
```

Avec `AuthorizeView` :

```xml
<AuthorizeView Policy="LetterPolicy">
    
    <Authorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Secondary">
            J'ai une lettre 'a' dans mon prénom
        </MudText>
    </Authorized>
    
    <NotAuthorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Secondary">
            Je n'ai pas de 'a' dans mon prénom
        </MudText>
    </NotAuthorized>
    
</AuthorizeView>
```



## Récupérer les `failures`

```cs
@code {
    [CascadingParameter] public Task<AuthenticationState>? StateTask { get; set; }

    AuthorizationFailure? failure;

    protected override async Task OnInitializedAsync()
    {
        if (StateTask is not null)
        {
            var state = await StateTask;

            var user = state.User;

            var authResult = await AuthService.AuthorizeAsync(user, "LetterPolicy");

            failure = authResult.Failure;
        }
    }
}
```

> On récupère le `User` grâce à `Task<AuthenticationState>`.
>
> Celui-ci est injecté automatiquement par `<AuthorizeRouteView>` :
> ```cs
> // Code de AuthorizeRouteView
> 
> protected override void Render(RenderTreeBuilder builder)
> {
>     if (ExistingCascadedAuthenticationState != null)
>     {
>         // If this component is already wrapped in a <CascadingAuthenticationState> (or another
>         // compatible provider), then don't interfere with the cascaded authentication state.
>         _renderAuthorizeRouteViewCoreDelegate(builder);
>     }
>     else
>     {
>         // Otherwise, implicitly wrap the output in a <CascadingAuthenticationState>
>         builder.OpenComponent<CascadingAuthenticationState>(0);
>         builder.AddComponentParameter(1, nameof(CascadingAuthenticationState.ChildContent), _renderAuthorizeRouteViewCoreDelegate);
>         builder.CloseComponent();
>     }
> }
> ```
>
> On voit que la sortie du routeur est enveloppée (Wrappée) avec `<CascadingAuthenticationState>`.

Et on les affiche :

```xml
<NotAuthorized>
    <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Secondary">
        Je n'ai pas de 'a' dans mon prénom
        <br/>
        @if (failure is not null)
        {
            <ul>
                @foreach (var reason in failure.FailureReasons)
                {
                    <li>@reason.Message</li>
                }
            </ul>
        }
    </MudText>
</NotAuthorized>
```

