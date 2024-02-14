# 09 Créer une `Custom Policy`

On voudrait créer une politique d'autorisation basée sur la date d'engagement.



## Ajouter la `Claim` pour gérer la date d'engagement

```cs
List<Claim> claims =
    [
        new Claim(ClaimTypes.Name, "admin"),
        new Claim(ClaimTypes.Email, "admin@gmail.com"),
        new Claim("Department", Credential.Department),
        new Claim("EmploymentDate", "2024-01-23") // <- ici
    ];
```



## Création D'un `Custom Requirement` : d'une condition personnalisée

Dans un dossier `Authorization` on va crée la classe `HRManagerProbationRequirement.cs` :

```cs
public class HRManagerProbationRequirement(int probationMonths) : IAuthorizationRequirement
{
    public int ProbationMonths { get; } = probationMonths;
}

public class HRManagerProbationRequirementHandler : AuthorizationHandler<HRManagerProbationRequirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, HRManagerProbationRequirement requirement)
    {
        var employmentDateClaim = context.User.FindFirst("EmploymentDate");

        if (DateTime.TryParse(employmentDateClaim?.Value, out var employmentDate) &&
            employmentDate.AddMonths(requirement.ProbationMonths) <= DateTime.Now) context.Succeed(requirement);

        return Task.CompletedTask;
    }
}
```



## Enregistrer le `Requirement`

`Program.cs`

```cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy(
        MyAuthPolicy.MustBelongToHRDepartment,
        policy => policy.RequireClaim("Department", "HR")
            .Requirements.Add(new HRManagerProbationRequirement(3))
    );
```

On doit injecter le `Handler` :

```cs
builder.Services.AddSingleton<IAuthorizationHandler, HRManagerProbationRequirementHandler>();
```



## Ajouter la page `HRDepartment` au menu

```html
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="" asp-page="/HRDepartment">
            HR Department
        </a>
    </li>
</ul>
```

