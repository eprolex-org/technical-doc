# 03. Service d'authentification par `cookie`



## Modification des adresses de `login` et `logout`

On peut gérer la route de login dans les options du service d'authentification par `cookie` :

```cs
builder.Services.AddAuthentication(
    	CookieAuthenticationDefaults.AuthenticationScheme
	)
    .AddCookie(options =>
        {
            options.LoginPath = "/Login";
            options.LogoutPath = "/Logout";
        }
    );
```





