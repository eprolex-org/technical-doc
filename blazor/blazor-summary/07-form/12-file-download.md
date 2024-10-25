# 12 File `Download`

Il faut un `endpoint` qui recherche et renvoie un `Results.File` et un bouton avec un lien vers un fichier particulier.



## Le endpoint

```cs
app.MapGet("/file/upload/{documentId}", (int documentId, DocumentRepository repo) =>
{
    var document = repo.GetDocumentById(documentId)
    
    var path = Path.Combine(
        "/Users/kms/Desktop/UploadedFileDesktop", document.FileName
    );
    
    // si on enregistre le code MIME 
    // return Results.File(path, document.MIME);
    return Results.File(path, "application/pdf");
});
```

L'implémentation de `Document` et `DocumentRepository` n'a pas d'importance ici.

> ###  ! Différent `Content-Type` différent comportement
>
> Pour un `application/pdf`, le navigateur ouvre une visionneuse, tandis que pour un document `docx` de type `application/vnd.openxmlformats-officedocument.wordprocessingml.document`, le navigateur propose un téléchargement :
>
> ### Pour un `PDF`
>
> <img src="assets/pour-pdf-ouverture-onglet.png" alt="pour-pdf-ouverture-onglet" />
>
> ### Pour un `docx`
>
> <img src="assets/pour-un-docx-ouverture-telechargement.png" alt="pour-un-docx-ouverture-telechargement" />
>
> Cela ouvre aussi un nouvel onglet (`_blank`), mais cette fois seulement en proposant de sauver le fichier sur son disque.





## Le bouton (en `MudBlazor`)

```react
<MudButton 
    Variant="Variant.Filled"
    Color="Color.Primary"
    Target="_blank"
    StartIcon="@Icons.Material.Filled.RemoveRedEye"
    Href="@($"/file/upload/{MyDocument.Id}")">
    Voire le fichier
</MudButton>                                           
```

```cs
@code {
	[Parameter] public Document? MyDocument { get; set; }
}   
```

