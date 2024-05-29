# DD Boîte à outils



## `Serialize`/`Deserialize`

```cs
public static class JsonExtensions
{
    public static byte[] Serialize(this object? obj)
    {
        if (obj is null) return Array.Empty<byte>();

        var options = new JsonSerializerOptions
        {
            WriteIndented = true,
            
        };

        var json = JsonSerializer.Serialize(obj, options);

        return Encoding.ASCII.GetBytes(json);
    }

    public static object? Deserialize<T>(this byte[] arrBytes)
    {
        var json = Encoding.Default.GetString(arrBytes);
        
        return JsonSerializer.Deserialize<T>(json);
    }
}
```

