## mud-09 `MudTextField`



## Utiliser `ValueChanged` plutôt que `@bind-Value`

Si on a de la logique à exécuter lorsque la valeur d'un champ `MudTextField` change, il faut utiliser `ValueChanged` plutôt que `@bind-Value`.

<img src="assets/errotr-with-value-changed.png" alt="errotr-with-value-changed" />

```cs
public void OnTelValueChanged(string telValue)
{
    // LOGIQUE   
}
```

Cependant cela génère une erreur car il faut préciser le `Type` du champ:

<img src="assets/type-precised-by-t-equal.png" alt="type-precised-by-t-equal" />