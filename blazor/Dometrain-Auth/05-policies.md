# 05 Policies

Une `Policy` es un ensemble de règles (`rules`), qui mises ensemble créé une nouvelle `règle`.



## `AddAuthorization`

<img src="assets/policy-built-in-to-us-rrfabbwhd.png" alt="policy-built-in-to-us-rrfabbwhd" />

```ruby
builder.Services.AddAuthorization(
    options => {
		options.AddPolicy(
            "RequireAdminRole", // policy name
            policy => {
            	policy.RequireRole("Admin");
            }
        );
    }
);
```

```ruby
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminAndManagerRoleOnly", policy =>
    {
        policy.RequireRole("Admin")
            .RequireRole("Manager");
    });
});
```



## Dans une `page` : `@attribute [Authorize]`

```ruby
@attribute [Authorize(Policy = "AdminAndManagerRoleOnly")]
```



## Créer son propre `Requirement`

```ruby
public class PassRequirement : IAuthorizationRequirement { }

public class PassSupervisorRequirementHandler(UtilisateurPassRepositoy repo) : AuthorizationHandler<PassRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        PassRequirement requirement
    )
    {
        var id = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        if(id is null) context.Fail();

        if (Guid.TryParse(id, out var guid))
        {
            var passes = repo.GetPasses();
            var utisateurPasses = passes.Where(p => p.UtilisateurId == guid);

            if (utisateurPasses.Any(p => p.Role == "Supervisor"))
            {
                context.Succeed(requirement);
                
                return Task.CompletedTask;
            }
        }
        
        context.Fail(new AuthorizationFailureReason(this, "une première raison"));
        context.Fail(new AuthorizationFailureReason(this, "une deuxième raison"));

        return Task.CompletedTask;
    }
}
```

L'utiliser dans `Program.cs`

```ruby
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SupervisorOnly", policy =>
    {
        policy.Requirements.Add(new PassRequirement());
    });
});
```

```ruby
builder.Services.AddTransient<IAuthorizationHandler, PassSupervisorRequirementHandler>();
```



### Utilisation dans une `page`

```ruby
@attribute [Authorize(Policy = "SupervisorOnly")]
```



#### `IAuthorizationService`  et  `AuthenticationStateProvider`

Plutôt que de permettre ou d'interdire l'entièreté de la `page`, on peut utiliser `IAuthorizaionService` pour contrôler l'accès aux resources :

```ruby
@inject IAuthorizationService AuthorizationService
@inject AuthenticationStateProvider AuthenticationState

@if (result is not null && result.Succeeded)
{
    <p>Secret for Supervisor</p>
}
else
{
    @foreach (var failure in result?.Failure?.FailureReasons)
    {
        <p>@failure.Message</p>
    }
}
    

@code {
    AuthorizationResult? result;

    protected override async Task OnInitializedAsync()
    {
        var state = await AuthenticationState.GetAuthenticationStateAsync();

        result = await AuthorizationService.AuthorizeAsync(state.User, "SupervisorOnly");   
    }
}
```

<img src="assets/authorization-faild-ytsrezadfffd.png" alt="authorization-faild-ytsrezadfffd" />

Ou bien si la personne peux y accéder :

<img src="assets/authorization-succede-uuiiffgdtheyusj.png" alt="authorization-succede-uuiiffgdtheyusj" />
