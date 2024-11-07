# BB. Générer un fichier



## `GenerateFile(path, sizeInKB)`

```cs
void GenerateFile(string filePath, long fileSizeInKiloBytes)
 {
     byte[] buffer = new byte[1024]; // 1 Ko
     new Random().NextBytes(buffer);

     using FileStream fs = new FileStream(filePath, FileMode.Create, FileAccess.Write);
     
     // Écrit 1 Ko de données aléatoires pour chaque Ko souhaité
     for (int i = 0; i < fileSizeInKiloBytes; i++)
     {
         fs.Write(buffer, 0, buffer.Length);
     }
 }
```



## Utilisation

```cs
const string FileGeneratedPath = "/Users/kms/Desktop/FileGenerated";

if (!Directory.Exists(FileGeneratedPath))
{
    Directory.CreateDirectory(FileGeneratedPath);
}

int sizeInKiloBytes = 1024 * 23; // 23 Mo (KB)

string fileName = $"{sizeInKiloBytes}_Ko_{new Random().Next(1, 100)}.pdf";

string fp = Path.Combine(FileGeneratedPath, fileName);


GenerateFile(fp, sizeInKiloBytes);

```

