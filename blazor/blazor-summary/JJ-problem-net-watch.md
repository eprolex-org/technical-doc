# JJ Problème avec `dotnet watch`

## Activer plus de log pour `watch`

dans `program.cs`

```cs
builder.Logging.SetMinimumLevel(LogLevel.Debug);
```



## Modifier l'arborescence

Après avoir modifier la structure de mon projet en changeant de place plusieurs composant, `dotnet watch` a cessé de mettre à jour les écrans.

```bash
dotnet watch ⌚ File changed: /Users/kms/Documents/GitHub/Eprolex/EprolexUI/ComponentLib/Component/DemandeAvis/DemandeConjointe/EpMandatForm.razor.
dotnet watch ⌚ No hot reload changes to apply.
```

Malgré une modification du `html` d'un composant je pouvait lire dans les `logs` qu'il n'y avait pas de `hot reload` à appliquer.

Le problème venait de la réécriture automatique du `.csproj` par `Rider` :

```xml
<ItemGroup>
  <UpToDateCheckInput Remove="Component\DemandeAvis\Form\Edit\DemandeConjointe\EpDemandeConjointe.razor" />
  <UpToDateCheckInput Remove="Component\DemandeAvis\Form\Edit\DemandeConjointe\EpMandatForm.razor" />
  <UpToDateCheckInput Remove="Component\DemandeAvis\Form\Edit\DemandeConjointe\EpMandatList.razor" />
  <UpToDateCheckInput Remove="Component\DemandeAvis\Form\Edit\DemandeConjointe\EpTitreConjointForm.razor" />
  <UpToDateCheckInput Remove="Component\DemandeAvis\Form\Edit\DemandeConjointe\EpTitreConjointItem.razor" />
  <UpToDateCheckInput Remove="Component\DemandeAvis\Form\Edit\DemandeConjointe\EpTitreConjointList.razor" />
</ItemGroup>

<ItemGroup>
  <AdditionalFiles Include="Component\DemandeAvis\DemandeConjointe\EpDemandeConjointe.razor" />
  <AdditionalFiles Include="Component\DemandeAvis\DemandeConjointe\EpMandatForm.razor" />
  <AdditionalFiles Include="Component\DemandeAvis\DemandeConjointe\EpMandatList.razor" />
  <AdditionalFiles Include="Component\DemandeAvis\DemandeConjointe\EpTitreConjointForm.razor" />
  <AdditionalFiles Include="Component\DemandeAvis\DemandeConjointe\EpTitreConjointItem.razor" />
  <AdditionalFiles Include="Component\DemandeAvis\DemandeConjointe\EpTitreConjointList.razor" />
</ItemGroup>
```

Une fois ces lignes supprimées, le `hot reload` c'est remis à fonctionner correctement :

```bash
dotnet watch ⌚ File changed: /Users/kms/Documents/GitHub/Eprolex/EprolexUI/ComponentLib/Component/DemandeAvis/DemandeConjointe/EpMandatList.razor.
dotnet watch 🔥 Hot reload of changes succeeded.
```

