## mud-10 `File Upload`

## File upload corrigé pour un seul fichier

```cs
@inject ISnackbar Snackbar

<MudStack Style="width: 100%">
    <MudFileUpload T="IBrowserFile" OnFilesChanged="OnInputFileChanged" AppendMultipleFiles="false" Hidden="false" Class="flex-1" InputClass="absolute mud-width-full mud-height-full overflow-hidden z-20" InputStyle="opacity:0"
                   @ondragenter="@SetDragClass" @ondragleave="@ClearDragClass" @ondragend="@ClearDragClass">
        <ButtonTemplate>
            <MudPaper Height="300px" Outlined="true" Class="@DragClass">
                <MudText Typo="Typo.h6">Drag and drop files here or click</MudText>
               
                    @if(fileName != "") 
                    {
                        <MudChip Color="Color.Dark" Text="@fileName" />
                    }
                
            </MudPaper>
        </ButtonTemplate>
    </MudFileUpload>
    <MudToolBar DisableGutters="true" Class="gap-4">
        <MudButton OnClick="Upload" Disabled="@(fileName == "")" Color="Color.Primary" Variant="Variant.Filled">Upload</MudButton>
        <MudButton OnClick="Clear" Disabled="@(fileName == "")" Color="Color.Error" Variant="Variant.Filled">Clear</MudButton>
    </MudToolBar>
</MudStack>

@code {
    private static string DefaultDragClass = "relative rounded-lg border-2 border-dashed pa-4 mt-4 mud-width-full mud-height-full z-10";
    private string DragClass = DefaultDragClass;
    private string fileName = "";

    private void OnInputFileChanged(InputFileChangeEventArgs e)
    {
        ClearDragClass();
        var file = e.GetMultipleFiles()[0];
        
        fileName = file.Name;
    }

    private async Task Clear()
    {
        fileName = "";
        ClearDragClass();
        await Task.Delay(100);
    }
    private void Upload()
    {
        //Upload the files here
        Snackbar.Configuration.PositionClass = Defaults.Classes.Position.TopCenter;
        Snackbar.Add("TODO: Upload your files!", Severity.Normal);
    }

    private void SetDragClass()
    {
        DragClass = $"{DefaultDragClass} mud-border-primary";
    }

    private void ClearDragClass()
    {
        DragClass = DefaultDragClass;
    }
}
```

Reste le bug qu'on ne peut pas remettre un fichier que l'on vient de `clear`.

https://github.com/MudBlazor/MudBlazor/issues/6699