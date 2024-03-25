# 08 `TaskCompletionSource`

Permet de construire des `Task` en dehors de celles existantes dans `.net`.

C'est un `Wrapper` pour des actions asynchrone nous permettant de renvoyer une `Task`.

```cs
Task<bool> IsTooLongWordAsync(string wordToTest)
{
    var tcs = new TaskCompletionSource<bool>();

    Task.Delay(TimeSpan.FromSeconds(2)).Wait();


    switch (wordToTest.Length)
    {
        case 0 or > 12:
            tcs.SetException(new Exception("Invalid length"));
            break;
        case < 6:
            tcs.SetResult(false);
            break;
        case >= 6:
            tcs.SetResult(true);
            break;
    }

    return tcs.Task;
}
```

La `Task` n'est pas résolu avant qu'un appelle à `SetResult` ne soit fait.

`SetException` va lancer une `Exception`.

Il existe aussi `TrySetResult` et `TrySetException` qui renvoient un `boolean`.

C'est une `factory` de `Task` à effectuer.

On renvoie la `Task` avec `tcs.Task`.



Programme pour tester :

```cs
Console.WriteLine("Hello, Task Completion Source!");

var word = "example";
Console.WriteLine($"Is the word {word} too long ? : {await IsTooLongWordAsync(word)}"); // true

word = "two";
Console.WriteLine($"Is the word {word} too long ? : {await IsTooLongWordAsync(word)}"); // false

word = "";
Console.WriteLine($"Is the word {word} too long ? : {await IsTooLongWordAsync(word)}"); // BOUM
```

