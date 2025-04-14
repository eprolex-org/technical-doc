# 04 `Logout`

```cs
app.MapPost("/logout", ([FromForm] string? returnUrl, HttpContext) => {
    // Vérifier plus précisément en prod returnUrl cf 03-login
    if (string.IsNullOrEmpty(returnUrl) 
        || !Uri.IsWellFormedUriString(returnUrl, UriKind.Relative))
    {
        returnUrl = "/";
    }
    
    var properties = new AuthenticationProperties { RedirectUri = returnUrl };
    var entraIdScheme = "entraIdScheme";
    var cookieScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    
    return TypedResults.Signout(
        properties, [entraIdSheme, cookieScheme]
    );
    
});
```

