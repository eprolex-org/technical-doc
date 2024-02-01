# 03. Création d'une page de `Login`



## Mise en place

> Les `razor page` sont basées sur `MVC` mais applique le pattern `MVVM`.

On créé un dossier `Account` dans le dossier `Pages` et dedans deux fichier :

`Login.cshtml` et `Login.cshtml.cs`

```html
@page
@model SecurityFranckLiu.Pages.Account.Login
@{
    ViewData["Title"] = "Login";
}

<h1>@ViewData["Title"]</h1>
    
<div class="text-danger pa-2" asp-validation-summary="All"></div>

<form method="post">
 
    <label asp-for="Credential.UserName"></label>
    <input class="form-control" type="text" asp-for="Credential.UserName" />
	<span class="text-danger" asp-validation-for="Credentila.UserName"></span>

    
    <label asp-for="Credential.Password"></label>
    <input class="form-control" type="text" asp-for="Credential.Password" />
    <span class="text-danger" asp-validation-for="Credentila.Password"></span>
        
    <input type="submit" class="btn btn-primary" value="login" />
</form>
```

> `asp-validation-summary="ModelOnly"` ne fonctionne pas chez moi.
>
> Utilisation de `asp-validation-summary="All"` à la place.

```cs
public class Login : PageModel
{
    [BindProperty]
    public Credential Credential { get; set; }
    
    public void OnGet()
    {
    }
    
    publc vooid OnPost()
    {
        
    }
}

public class Credential
{
    [Required]
    [Display(Name = "User Name")]
    public string UserName { get; set; }
    [Required] 
    [DataType(DataType.Password)]
    public string Password { get; set; }
}
```

