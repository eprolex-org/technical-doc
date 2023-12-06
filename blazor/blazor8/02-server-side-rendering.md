# 02 `SSR` Server Side Rendering

## Créer un projet

En `CLI`

```bash
dotnet new blazor -o BlazorApp -int none
```

`-int` : interactivity Si à `none` on est en `SSR`.

<img src="assets/program-cs-modification.png" alt="program-cs-modification" style="zoom:67%;" />

### `builder.Services.AddRazorComponents();`

### `app.MapRazorComponents<App>();`

C'est une application `Web .Net` et on peut très bien y ajouter un `endpoint` par exemple :

```cs
// ...
app.MapGet("/hello", () => "hello world");

app.MapRazorComponents<App>();

app.Run();
```

<img src="assets/api-with-razor-page.png" alt="api-with-razor-page" style="zoom:67%;" />



## `Interctivity`

L'`interactivity` étant mise sur `none`, un simple bouton ne fonctionne pas.

```ruby
<p>@_clickNumber</p>
<p>
    <button @onclick="HandleClick">Click Me</button>
</p>

@code {
    private int _clickNumber = 1;
    
    private void HandleClick() => _clickNumber += 1; 
}
```

<img src="assets/interactivity-equal-none.png" alt="interactivity-equal-none" style="zoom:67%;" />

Comme attendu rien ne se passe.
