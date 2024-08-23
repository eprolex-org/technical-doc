# 01 Créer un `nuget package`

## Commande `pack`

Les éléments seront placés dans un fichier `.nupkg`, c'est un fichier `zip`.

```bash
dotnet pack -o ~/Desktop/MyPackages
```

J'obtiens un fichier `Model.1.0.0.nupkg`.

Je peux modifier son extension en `.zip` et le dézipper pour regarder dedans :

```bash
mv Model.1.0.0.nupkg Model.1.0.0.zip 
```

```bash
unzip Model.1.0.0.zip 
```

je peux voire l'arborescence du dossier :

```bash
tree

.
├── Model.1.0.0.zip
├── Model.nuspec
├── [Content_Types].xml
├── _rels
├── lib
│   └── net8.0
│       └── Model.dll
└── package
    └── services
        └── metadata
            └── core-properties
                └── 5aa800919e224993bf222b2196e53ad1.psmdcp
```

C'est le fichier `Model.nuspec` qui va contenir la description du `package`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <metadata>
    <id>Model</id>
    <version>1.0.0</version>
    <authors>Model</authors>
    <description>Package Description</description>
    <repository type="git" />
    <dependencies>
      <group targetFramework="net8.0" />
    </dependencies>
  </metadata>
</package>
```

On peut spécifier certaine info par exemple la `version` et l'`auteur` dans `.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net8.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
        <PackageId>UniqueID</PackageId>
        <Version>0.9.0</Version>
        <Authors>KMS</Authors>
        <Company>Council Of State</Company>
        <Product>Eprolex</Product>
    </PropertyGroup>

</Project>
```

J'obtiens maintenant cette arborescence :

```bash
tree

├── UniqueID.0.9.0.zip
├── UniqueID.nuspec
├── [Content_Types].xml
├── _rels
├── lib
│   └── net8.0
│       └── Model.dll
└── package
    └── services
        └── metadata
            └── core-properties
                └── aebc236b39f34bf3a71fd04082001b0a.psmdcp
```

On remarque que `UniqueId` est devenu le label du `package` et que le numéro de `version` n'est plus celui par défaut.

et ce fichier `.nuspec` :

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <metadata>
    <id>UniqueID</id>
    <version>0.9.0</version>
    <authors>KMS</authors>
    <description>Package Description</description>
    <repository type="git" />
    <dependencies>
      <group targetFramework="net8.0" />
    </dependencies>
  </metadata>
</package>
```

`UniqueId` est surtout utile pour `nuget.org` pour éviter les conflit.

De même la description est utilisé par `nuget.org`.

> ## `Release` mode
>
> Pour créer un `package` pour une `release`, on utilise `-c Release` :
>
> ```bash
> dotnet pack -o ~/Desktop/MyPackagesTwo -c Release
> ```
>
> ?? je ne sais pas ce que ça fait ??