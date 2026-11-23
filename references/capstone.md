# Capstone (.NET Framework 4.5 / C# 5.0 / VS 2013)

**Não há UM projeto final único.** Esta skill constrói **três
aplicações desktop separadas**, em ordem crescente de complexidade, cada uma com sua
própria solution. Não há solution única integrando as três.

## Ordem de construção / entregáveis

1. **Notepad** — WPF+MVVM puro, sem banco nem rede.
2. **CadCliente** — acrescenta ADO.NET/DAO + SQL Server sobre a mesma base MVVM.
3. **Chat** — em três partes: arquitetura (threads, sockets, protocolo) + layouts;
   `ChatCommon` + lado servidor; lado cliente.

Entregável de cada parte: a aplicação funcional + código-fonte. Stack de todos os
projetos: VS Express 2013 for Windows Desktop, .NET Framework 4.5.

## 1 — Editor de texto (`Notepad`)

Clone do Bloco de Notas do Windows, WPF + MVVM. Solution
`Notepad/Notepad.sln`, projeto único `Notepad` (`Notepad.csproj`).

- **Funcionalidades:** menus Arquivo (Novo, Abrir, Salvar, Salvar Como, Sair), Editar
  (Recortar, Copiar, Colar, Selecionar Tudo — via `ApplicationCommands`), Formatar
  (quebra automática de linha), Ajuda → Sobre; barra de status com
  linha e coluna da posição atual; **persiste preferências** (posição/tamanho da janela +
  `LineWrap`) em Settings — ao reabrir, restaura tudo.
- **Estrutura (pastas literais):** `Model/Content.cs` (único model: `Text` + `FileInfo`;
  `Load` via `File.ReadAllText`, gravação via `File.WriteAllText`);
  `ViewModel/ContentViewModel.cs`; `View/MainWindow.xaml` (+ `.xaml.cs`) e
  `View/AboutWindow.xaml`; `ApplicationService/IWindowServices.cs` (View implementa;
  ViewModel só conhece a interface); `Converter/BoolToLineWrapConverter.cs` e
  `Converter/BoolToScrollBarVisibilityConverter.cs`; `Lib/Mvvm.dll` referenciada
  (mini-framework próprio: `Bindable` + `Command`).
- **Padrão de comandos:** `ContentViewModel` expõe `NewFile`, `OpenFile`, `SaveFile`,
  `SaveAsFile`, `Exit`, `ShowAboutDialog` (via `IWindowServices`: Open/SaveFileDialog);
  título da janela via binding (`WindowTitle` mostra `FileInfo.Name`); fechamento
  controlado por field `CloseWindow` (evita sair sem salvar/gravar estado).

## 2 — Cadastro de clientes (`CadCliente`)

CRUD WPF + MVVM + banco de dados. Solution `CadCliente/CadCliente.sln`,
projeto único `CadCliente` (`CadCliente.csproj`).

- **Funcionalidades:** lista de clientes à esquerda (ListBox + pesquisa por nome,
  `LIKE %padrão%`), formulário à direita (Nome, E-mail, Telefone; Logradouro, Número,
  Complemento, CEP); menus Arquivo (Sair), Editar, Cliente (Novo, Excluir, Editar,
  Gravar, Cancelar, Pesquisar); botões com habilitação por estado; confirmação antes
  de excluir; dados persistem ao fechar/reabrir.
- **Banco:** `clientedb`, SQL Server local, instância `SQLEXPRESS`,
  `Integrated Security=True` — connection string literal em `App.config`:
  `Data Source=(local)\SQLEXPRESS;Initial Catalog=clientedb;Integrated Security=True`,
  chave `dbConnString`, provider `System.Data.SqlClient` (chave `dbProvider`). Tabelas
  `Cliente` (Id, Nome, Email, Telefone — só Email/Telefone aceitam nulo) e `Endereco`
  (ClienteID = mesmo Id do cliente, 1:1; Logradouro, Número, Complemento, CEP).
  **Id NÃO é identity:** gerado na mão por `LerMaiorId` (`SELECT MAX(Id) FROM Cliente`,
  0 se vazio) + 1, porque o endereço precisa do mesmo Id.
- **Estrutura (pastas literais):** `Model/Cliente.cs` e `Model/Endereco.cs` (herdam
  `Bindable`; `Id` é `int?` — null = CREATE, não-null = UPDATE; validação "não pode
  ser vazio" em Nome, Logradouro e Número; `Cliente.Endereco` nunca retorna null —
  cria sob demanda); `Model/Db/ClienteDao.cs` (um só DAO para Cliente+Endereço, pois
  endereço "não tem vida própria"); `ViewModel/ClienteViewModel.cs` (lista `Clientes`,
  `SelectedIndex`, `UpdateBindings`, `ConfirmSave`/`ConfirmDelete`);
  `View/MainWindow.xaml`; `ApplicationService/IWindowServices.cs`;
  `Lib/Dao.dll` + `Lib/Mvvm.dll` referenciadas.
- **DAO (`ClienteDao`, herda `DAO`, via `DAOFactory`):** `ListarClientes(padrão)`
  (SELECT com `INNER JOIN` Cliente+Endereco, `WHERE Nome LIKE`, `ORDER BY Nome`, itera
  `DbDataReader.Read()` + `ExtrairDados` com checagem `DBNull`); `LerCliente(id)`
  (popula o objeto passado, não retorna um novo); `Inserir` (transação
  `BeginTransaction` + `Commit`: `INSERT INTO Cliente` + `INSERT INTO Endereco`);
  `Alterar` (transação, `UPDATE` nas duas tabelas); `Excluir(id)` (transação, apaga
  **primeiro Endereco, depois Cliente**, por causa da FK); `AddParameter` (null C# →
  `DBNull.Value`); `LerMaiorId` e `ExtrairDados` (privados, auxiliares).

## 3 — Chat Cliente-Servidor (`Chat`)

Chat multiusuário TCP. Solution `Chat/Chat.sln` com **3 projetos**:
`ChatServer` (WPF), `ChatClient` (WPF) e `ChatCommon` (biblioteca/DLL com o protocolo).

- **Funcionamento:** servidor escuta na **porta 3000** (configurável; `MinPort 1000`,
  `MaxPort 20000` em `App.config`); cliente faz login (Servidor=`localhost`, Porta,
  Nome) e entra na sala; lista de membros à direita, mensagens à esquerda, Enviar por
  botão ou Enter; entrada/saída de membros anunciada a todos; desconectar o servidor
  derruba todos os clientes.
- **Modelo de threads:** thread principal do servidor só aguarda conexões;
  **uma thread por cliente conectado** (10 clientes = 10 threads); cada `Member` tem
  seu socket + `InputHandler`/`OutputHandler`.
- **`ChatCommon` (4 classes literais):** `ChatCommand.cs` (comando genérico:
  `ChatCommandType` + parâmetro; `ToString()` = `Tipo|Parâmetro`; `Parse()` estático
  faz o inverso), `CommandEventArgs.cs`, `InputHandler.cs` (thread própria em loop
  lendo o canal e disparando **um evento por tipo de mensagem**), `OutputHandler.cs`
  (só escreve no canal via `StreamWriter` com `AutoFlush=true`; métodos
  `SendEnterRoomCommand`, `SendGetMembersCommand`, etc. — um por tipo de comando).
- **Protocolo (`ChatCommandType`, 11 tipos):** `EnterRoom`, `EnterRoomResponse`
  (`OK` ou texto de erro), `GetMembers`, `GetMembersResponse`, `SendMessage`,
  `MessageReceived`, `MemberEntered`, `MemberLeaving`, `MemberLeft`, `MemberCanLeave`,
  `ServerDisconnecting`.
- **`ChatServer`:** `Model/ChatServer.cs` + `Model/Member.cs` (cada `Member` = um cliente
  conectado, com thread, socket, handlers e `HandleMemberInteraction`);
  `ViewModel/ServerViewModel.cs`; `View/ServerWindow.xaml` (grid 2×2: Porta, botões
  Conectar/Desconectar).
- **`ChatClient`:** `Model/ChatClient.cs` (field `TcpClient` + `InputHandler`/
  `OutputHandler`; `Connect(servidor, porta)` com `SocketException` tratada;
  `Disconnect()` = só `TcpClient.Close()`); `ViewModel/LoginViewModel.cs` e
  `ViewModel/ChatViewModel.cs`; `View/LoginWindow.xaml` e `View/ChatWindow.xaml`;
  `ApplicationService/IChatWindowServices.cs` (`RunOnUiThread` via `Dispatcher.Invoke`,
  `ScrollMessagesToEnd`, `CloseWindow`) e `ILoginWindowServices.cs`. Referencia
  `ChatCommon` (ProjectReference) + `Lib/Mvvm.dll`.
- **Detalhe de UX:** o binding de `Message` só atualiza em `LostFocus`,
  então o Enter é tratado no code-behind: move o foco para o botão Enviar (força o
  update), chama `Send()`, devolve o foco à caixa. Fechar pelo X cancela o fechamento
  (`e.Cancel = true`) e chama `LeaveRoom()` — só fecha de fato após o servidor
  responder `MemberCanLeave`.

## O que o capstone usa das partes anteriores

| Parte anterior | Uso no capstone |
|---|---|
| Operadores e Casting | Casts; `int?` para Id nulo no CadCliente |
| Delegates e Events | Eventos do `InputHandler`; `Command` do framework MVVM |
| Assemblies | `Mvvm.dll` e `Dao.dll` referenciadas da pasta `Lib/` |
| LINQ | **Não aparece** — o DAO usa SQL puro + `DbDataReader` |
| XML | **Não aparece** como `XmlDocument`/XPath |
| ADO.NET | **CadCliente inteiro**: `DAO`/`DAOFactory`, `DbDataReader`, parâmetros, `BeginTransaction`/`Commit`, `DBNull` |
| Entity Framework | **Não usado** — acesso a dados é ADO.NET puro |
| WPF Parte 1 | XAML, `DockPanel`/`Grid`/`StackPanel`, menus, `ListBox`/`TextBox` |
| WPF Parte 2 / MVVM | **Base de tudo**: `Model`/`View`/`ViewModel`, `Bindable`, `Command`, binding, converters, validação, Settings, `IWindowServices` |
| Multithread | Chat: thread por cliente + thread de escuta do `InputHandler` |
| Assíncrona e Paralela | `LoginViewModel.ConnectAsync` (`async`/`await`) para não travar a UI |
| Serialização | **Não usada** nas apps (persistência via `File.Read/WriteAllText`, SQL Server, Settings) |
| Programação em Rede | Chat: socket TCP (`TcpClient`), porta 3000, protocolo texto `Tipo\|Parâmetro` |

## Serialização XML — exemplo literal

Exemplo de serialização XML.
Literal de `Serializacao-XML/Program.cs`
(projeto console, .NET Framework 4.5; verificado por compilação — 0 erros):

```csharp
using System;
using System.IO;
using System.Xml.Serialization;

class Program
{
    static void Main()
    {
        Serialize();
        Deserialize();
    }

    static void Serialize()
    {
        Esporte e = new Esporte();
        e.Nome = "Judô";
        e.Individual = true;

        Atleta a = new Atleta();
        a.Nome = "José Silva";
        a.Pais = "Brasil";
        a.Esporte = e;

        XmlSerializer ser = new XmlSerializer(typeof(Atleta));
        using (FileStream fs = File.OpenWrite("atleta.xml"))
        {
            ser.Serialize(fs, a);
        }

    }

    static void Deserialize()
    {
        XmlSerializer ser = new XmlSerializer(typeof(Atleta));
        using (FileStream fs = File.OpenRead("atleta.xml"))
        {
            Atleta a = (Atleta)ser.Deserialize(fs);
            Console.WriteLine(a);
        }
    }
}

[XmlRoot(ElementName = "AtletaEsporte")]
public class Atleta
{
    public string Nome { get; set; }

    [XmlAttribute]
    public string Pais { get; set; }

    public Esporte Esporte { get; set; }

    public override string ToString()
    {
        return string.Format("{0}, {1}, {2}", Nome, Pais, Esporte);
    }
}

public class Esporte
{
    public string Nome { get; set; }

    [XmlElement(ElementName = "Ind")]
    public bool Individual { get; set; }

    public override string ToString()
    {
        return string.Format("{0}, {1}", Nome, Individual);
    }
}
```

**Quando serializar:** serializar é transformar um objeto em dados de
um formato para **guardar o estado e recuperá-lo depois** — destinos: memória, arquivo
ou rede; quem decide o destino é a **stream** (`FileStream`, sempre em `using`).
Serializar um objeto serializa automaticamente suas dependências (grafo). `[NonSerialized]`
quando: a informação não é necessária na recuperação, não é possível serializar, ou
para economizar espaço. Coleções (`List<Aluno>`) serializam todos os itens juntos.

**XML vs. binário (só a coluna XML vale aqui):**

| | XML (`XmlSerializer`, `System.Xml.Serialization`) |
|---|---|
| `[Serializable]` | **Não precisa** |
| Visibilidade | Só **public** |
| Construtor | Classes precisam de **construtor padrão** |
| Construtor do serializador | `new XmlSerializer(typeof(Atleta))` — **o tipo é obrigatório** |
| Métodos | `Serialize(stream, obj)` / `Deserialize(stream)` (retorna `object` → cast) |
| Saída | Documento XML legível; tag raiz = nome da classe, uma tag por property |

Customização: `[XmlRoot(ElementName = "...")]`, `[XmlElement(ElementName = "...")]`,
`[XmlAttribute]`, `[XmlText]`. Atenção: **vale o código — a raiz é `AtletaEsporte`** (sem espaço).
