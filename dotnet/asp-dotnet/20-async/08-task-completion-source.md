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

`TaskCompletionSource` permet de créer une `Task` et de contrôler son cycle de vie.



## `TaskCreationOptions.RunContinuationAsynchronously`

### Code de test

```cs
Console.WriteLine($"Thread in Program start : {Thread.CurrentThread.ManagedThreadId}");

TriggerEvent();
Console.WriteLine($"Thread after TriggerEvent : {Thread.CurrentThread.ManagedThreadId}");

var result = await DoSomethingAsync();
Console.WriteLine($"Thread after DoSomethingAsync : {Thread.CurrentThread.ManagedThreadId}");

Console.WriteLine($"result: {result}");

Console.WriteLine($"Thread in Program stop : {Thread.CurrentThread.ManagedThreadId}");

async Task TriggerEvent()
{
Console.WriteLine($"Thread in TriggerEvent Start : {Thread.CurrentThread.ManagedThreadId}");
    await Task.Delay(TimeSpan.FromSeconds(4));
    EventWrapper.Trigger();
Console.WriteLine($"Thread in TriggerEvent Stop : {Thread.CurrentThread.ManagedThreadId}");
}

Task<int> DoSomethingAsync()
{
    Console.WriteLine($"Thread in DoSomethingAsync Start : {Thread.CurrentThread.ManagedThreadId}");
    var tcs = new TaskCompletionSource<int>(TaskCreationOptions.RunContinuationsAsynchronously);
    // var tcs = new TaskCompletionSource<int>();
    
    EventWrapper.MyEvent += (sender, args) =>
    {
        Console.WriteLine($"Thread before set result: {Thread.CurrentThread.ManagedThreadId}");
        Thread.Sleep(TimeSpan.FromSeconds(2));

        tcs.SetResult(2);
        Console.WriteLine($"Thread after set result: {Thread.CurrentThread.ManagedThreadId}");
    };
    
    Console.WriteLine($"Thread in DoSomethingAsync before Return : {Thread.CurrentThread.ManagedThreadId}");
    return tcs.Task;
}

delegate void MyEventHandler(object sender, EventArgs ea);

static class EventWrapper
{
    public static event MyEventHandler MyEvent;
    
    public static void Trigger() => MyEvent.Invoke(null, EventArgs.Empty);
}
```



### Résultats

Avec :

```cs
var tcs = new TaskCompletionSource<int>();
```

```
Thread in Program start : 1
Thread in TriggerEvent Start : 1
Thread after TriggerEvent : 1
Thread in DoSomethingAsync Start : 1
Thread in DoSomethingAsync before Return : 1
Thread before set result: 4
Thread after DoSomethingAsync : 4
result: 2
Thread in Program stop : 4
Thread after set result: 4
Thread in TriggerEvent Stop : 4
```

Avec :

```cs
var tcs = new TaskCompletionSource<int>(TaskCreationOptions.RunContinuationsAsynchronously);
```

```
Thread in Program start : 1
Thread in TriggerEvent Start : 1
Thread after TriggerEvent : 1
Thread in DoSomethingAsync Start : 1
Thread in DoSomethingAsync before Return : 1
Thread before set result: 4
Thread after set result: 4
Thread in TriggerEvent Stop : 4
Thread after DoSomethingAsync : 9
result: 2
Thread in Program stop : 9
```

On voit ici qu'avec `RunContinuationAsynchronously`, une `Thread` est créée lors du retour de la continuation :

```cs
var result = await DoSomethingAsync(); // + Thread 9
```

Ce qui n'est pas le cas sans cette option. À voire si cela peut jouer sur les performance en pratique.
