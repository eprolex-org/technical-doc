# 04 Authorization avec `ASP.NET`

## Mise en place

```cs
app.UseAuthorization();
```



## Ajouter les règles d'`authorization`

On doit ajouter des `policies` au service `Authorization` :

```cs
builder.Services.AddAuthorization(builder => {
    builder.AddPolicy
});
```

