# WPF + MVVM (.NET Framework 4.5 / C# 5.0 / VS 2013)

WPF com MVVM no estilo clássico. Definição: "MVVM significa Model
View ViewModel. É uma forma de estruturar aplicações WPF em três camadas" —
View = XAML + code-behind; Model = lógica de negócio, sem nenhuma dependência da View;
ViewModel = intermediador. Regra explícita: "A view pode referenciá-lo, mas ele não
pode referenciar a view".

**Framework MVVM externo: nenhum.** É MVVM puro/hand-rolled: esta skill define o próprio
mini-framework (`ClassicDesktop.Mvvm`, Class Library com só 2 classes — `Bindable` e
`Command`), referenciado por `ProjectReference`. Não há Prism, MVVM Light,
Caliburn.Micro ou NuGet em nenhum projeto desta skill.

## Estrutura do exemplo MVVM

Solution `Mvvm/Mvvm.sln`, 2 projetos:
- `Mvvm` (WinExe): pastas `Model/`, `View/`, `ViewModel/`, `ApplicationServices/`
- `MvvmFramework` (Class Library, namespace `ClassicDesktop.Mvvm`): `Bindable.cs`, `Command.cs`

Exemplo: lista de itens (strings) com adicionar/excluir.

**Model** — `Mvvm/Model/Itens.cs`: classe simples, sem WPF, só `List<string>` em
memória:

```csharp
class Itens
{
    List<string> nomes = new List<string>();
    public void Adicionar(string nome) { nomes.Add(nome); ShowDebug(); }
    public void Remover(int pos) { nomes.RemoveAt(pos); ShowDebug(); }
}
```

**ViewModel** — `Mvvm/ViewModel/ItensViewModel.cs`: herda `Bindable`; expõe `NomeItem`
(string), `Lista` (`ObservableCollection<string>`), `Indice` (int), `TextoIndice`
(string derivada) e dois comandos (`AdicionarCommand`, `ExcluirCommand`):

```csharp
AdicionarCommand = new Command(Adicionar, false);
ExcluirCommand = new Command(Excluir);
itens = new Itens();
Lista = new ObservableCollection<string>();
Indice = -1;
ErrorsChanged += (s, a) => AdicionarCommand.CanExecute = !HasErrors;
```

**DataContext — declarativo no XAML** (não no code-behind), com instância nomeada
(`Mvvm/View/MainWindow.xaml`):

```xml
<Window.DataContext>
    <vm:ItensViewModel x:Name="viewModel" />
</Window.DataContext>
```

com `xmlns:vm="clr-namespace:Mvvm.ViewModel"`. O code-behind acessa o ViewModel pelo
nome declarado no XAML (`viewModel`), não por cast de `DataContext`.

## `Bindable` — literal de `MvvmFramework/Bindable.cs`

Base de todo ViewModel/Model notificável. Implementa `INotifyDataErrorInfo` **e**
`INotifyPropertyChanged`:

```csharp
public abstract class Bindable : INotifyDataErrorInfo, INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;
    public event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;
    // ...
    protected void SetValue<T>(ref T property, T value, [CallerMemberName] string propertyName = null)
    {
        if (object.Equals(property, value)) { return; }
        property = value;
        OnPropertyChanged(propertyName);
    }
    protected void OnPropertyChanged(string propertyName)
    {
        if (PropertyChanged != null)
        {
            PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```

Uso típico no ViewModel — o setter de `Indice` dispara notificação de propriedade
derivada e habilita/desabilita comando:

```csharp
set
{
    SetValue(ref indice, value);
    OnPropertyChanged("TextoIndice");
    ExcluirCommand.CanExecute = indice >= 0;
}
```

## `Command` — literal de `MvvmFramework/Command.cs`

Não há `RelayCommand` com esse nome. Comando genérico por `Action`, com `CanExecute`
setável que dispara `CanExecuteChanged`:

```csharp
public class Command : ICommand
{
    Action commandAction;
    bool canExecute;
    public Command(Action commandAction, bool canExecute = true)
    { this.commandAction = commandAction; this.canExecute = canExecute; }
    bool ICommand.CanExecute(object parameter) { return canExecute; }
    public bool CanExecute
    {
        get { return canExecute; }
        set
        {
            if (canExecute != value)
            {
                canExecute = value;
                if (CanExecuteChanged != null)
                { CanExecuteChanged(this, EventArgs.Empty); }
            }
        }
    }
    public event EventHandler CanExecuteChanged;
    void ICommand.Execute(object parameter)
    { if (commandAction != null) { commandAction(); } }
}
```

Não usa `CommandManager.RequerySuggested` em nenhum dos dois comandos deste exemplo.

## Como a lista editável é feita

Não há "salvar em banco" aqui — a persistência é só memória +
`Debug.WriteLine`. O fluxo:

- **Adicionar:** digita no `TextBox` → `NomeItem` (TwoWay,
  `UpdateSourceTrigger=PropertyChanged`). O setter valida: se
  `string.IsNullOrWhiteSpace`, `AddError("Item em branco")`, senão `RemoveErrors()`.
  O `AdicionarCommand` nasce **desabilitado** (`new Command(Adicionar, false)`) e só
  habilita quando `ErrorsChanged` indica `!HasErrors`. `Adicionar()` grava no Model,
  na `ObservableCollection`, limpa o campo, reseta `Indice = -1` e devolve o foco via
  `WindowServices.SetFocus()`.
- **Excluir:** `ListBox.SelectedIndex` ↔ `Indice` (TwoWay). O setter faz
  `ExcluirCommand.CanExecute = indice >= 0` — botão desabilitado sem seleção.
  `Excluir()` remove do Model e da coleção, reseta `Indice = -1`.
- **Feedback:** `TextoIndice` (só-leitura, notificada manualmente) mostra "Nenhum item
  selecionado" ou "Índice selecionado: N" na `StatusBar`.
- **Edição inline de item existente: não há.** A lista só permite adicionar no fim e
  excluir o selecionado; não há edição do item nem comando de salvar. (O exemplo `BindingList`
  permite editar Nome/Idade/Brasileiro da pessoa selecionada nos `TextBox`,
  mas sem salvar — só demonstração de binding.)

**Fuga controlada do MVVM puro** — o ViewModel precisa devolver o foco ao `TextBox`
após adicionar, mas não pode referenciar a View. Solução: interface
`Mvvm/ApplicationServices/IWindowServices.cs`:

```csharp
interface IWindowServices { void SetFocus(); }
```

A janela implementa (`MainWindow : Window, IWindowServices`, `viewModel.WindowServices
= this;` no construtor). Justificativa: com o ViewModel independente da View, dá para
testá-lo com uma implementação falsa de `IWindowServices`.

## Padrões de binding no XAML (trechos literais)

**ElementName — controle para controle** (`BindingControl/MainWindow.xaml`):

```xml
<Label x:Name="label" Margin="10,0,10,0" Content="{Binding ElementName=slider, Path=Value}" FontSize="24" FontWeight="Bold" HorizontalAlignment="Center" />
<TextBox Text="{Binding ElementName=slider, Path=Value, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" Padding="3" Margin="10,10,10,0" />
```

**DataContext no container + `Path=`** (`BindingObject/MainWindow.xaml`):

```xml
<TextBox x:Name="txtNome" Grid.Row="0" Grid.Column="1" VerticalAlignment="Center" Padding="2" Text="{Binding Path=Nome}" />
<TextBox x:Name="txtIdade" Grid.Row="1" Grid.Column="1" VerticalAlignment="Center" Padding="2" Text="{Binding Path=Idade}" />
<CheckBox x:Name="chkBrasileiro" Grid.Row="2" Grid.Column="1" VerticalAlignment="Center" IsChecked="{Binding Path=Brasileiro}" />
```

**Coleção + item selecionado como DataContext** (`BindingList/MainWindow.xaml`):

```xml
<ListBox Name="list" Grid.Row="0" Grid.Column="0" Margin="5" DisplayMemberPath="Nome" />
<Grid Name="gridPessoa" Margin="5" DataContext="{Binding ElementName=list, Path=SelectedItem}">
```

**Conversão — `StringFormat` e `IValueConverter`** (`Conversion/MainWindow.xaml`):

```xml
<TextBox x:Name="txtAltura" Grid.Row="1" Grid.Column="1" VerticalAlignment="Center" Padding="2" Text="{Binding Path=Altura, StringFormat={}{0:N2}}" />
```

**Validação — `ValidationRule` no binding** (`Validation/MainWindow.xaml`):

```xml
<TextBox Name="txtId" FontSize="13" Padding="3" Validation.Error="OnValidationError">
    <TextBox.Text>
        <Binding Path="Id" UpdateSourceTrigger="PropertyChanged" NotifyOnValidationError="True">
            <Binding.ValidationRules>
                <local:SizeValidator MaxSize="10" />
            </Binding.ValidationRules>
        </Binding>
    </TextBox.Text>
</TextBox>
```

**Bindings da View final MVVM** (`Mvvm/View/MainWindow.xaml`):

```xml
<TextBox Name="txtItem" Grid.Row="0" Grid.Column="1" Text="{Binding NomeItem, UpdateSourceTrigger=PropertyChanged}" Padding="3" FontSize="14" VerticalAlignment="Center">
<Button Grid.Row="0" Grid.Column="2" Content="Adicionar" Width="70" Margin="5,0,0,0" Command="{Binding AdicionarCommand}" />
<TextBlock Text="{Binding TextoIndice}" />
<Button DockPanel.Dock="Bottom" Content="Excluir" HorizontalAlignment="Right" Width="60" Height="25" Margin="0,5,0,0" Command="{Binding ExcluirCommand}" />
<ListBox Name="listBox" ItemsSource="{Binding Lista}" SelectedIndex="{Binding Indice, Mode=TwoWay}" />
```

**Comando via `StaticResource`** (`Command/MainWindow.xaml`): um comando, dois
controles (menu + botão):

```xml
<Window.Resources>
    <c:ProcessarCommand x:Key="cmd" />
</Window.Resources>
<MenuItem Header="Processar" Padding="2" Command="{Binding Source={StaticResource cmd}}" />
<Button Content="Processar" Width="80" Height="40" Command="{Binding Source={StaticResource cmd}}" />
```

**Settings** (`Prefs/MainWindow.xaml`): preferências do usuário ligadas por
binding (`Title="{Binding Source={StaticResource Settings}, Path=Default.Title}"`).

**XAML estrutural sem binding:** `Grid` com `Auto`/`*`, `StackPanel`,
`DockPanel`, `GroupBox`, menus, `ListBox`/`TextBox`/`Button`. Primeira aplicação:
só `Grid` + `Button Click="Button_Click"` + `Label x:Name="texto"`.

## O que esta skill NÃO cobre

`DataTemplate`, `ItemsControl` além de `ListBox`, `CollectionView`/`ICollectionView`,
navegação entre janelas, `DialogService`, persistência real (o "Model" é memória
volátil). Controles usados: `Window`, `Grid`, `StackPanel`, `DockPanel`, `GroupBox`,
`Label`, `TextBox`, `Button`, `CheckBox`, `ListBox`, `Slider`, `Menu`/`MenuItem`,
`StatusBar`/`TextBlock`, `Border`, `AdornedElementPlaceholder`.
