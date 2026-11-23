---
name: csharp-classico-desktop
description: Use when writing classic .NET desktop code: C# language fundamentals, ADO.NET data access, WPF with MVVM, and a complete capstone — on .NET Framework 4.5 / C# 5.0, with the version limits stated.
---

# C# clássico desktop

Método completo de C# desktop clássico: fundamentos da linguagem, acesso a dados
com ADO.NET sem ORM, WPF com MVVM, e três aplicações finais. Tudo é
**.NET Framework 4.5 / C# 5.0 / Visual Studio 2013 / SQL Server Express local** —
a skill marca a versão em cada bloco e nunca apresenta nada disto como .NET atual.

## Stack clássica (datada, declarada)

| Camada | Versão clássica |
|---|---|
| Runtime | .NET Framework 4.5 (`TargetFrameworkVersion v4.5`, `ToolsVersion 12.0`) |
| Linguagem | C# 5.0 (sem `?.`, sem interpolação `$""`, sem file-scoped namespace, sem nullable) |
| IDE | Visual Studio 2013 (artefatos `.v12.suo`) |
| Banco | SQL Server Express local, instância `(local)\SQLEXPRESS`, autenticação integrada |
| WPF | .NET Framework (PresentationCore/Framework, `System.Xaml`) |

Detalhe completo de defasagem em [defasagem.md](references/defasagem.md).

## Fluxo

1. **Linguagem** — fundamentos, coleções, LINQ em query syntax ([linguagem-csharp.md](references/linguagem-csharp.md)).
2. **Dados** — ADO.NET genérico via `DbProviderFactory`, sem ORM ([ado-net.md](references/ado-net.md)).
3. **Interface** — WPF + MVVM com o mini-framework próprio desta skill ([wpf-mvvm.md](references/wpf-mvvm.md)).
4. **Capstone** — três aplicações em ordem crescente: Notepad, CadCliente, Chat ([capstone.md](references/capstone.md)).

## Regras duras

1. **Marque a versão.** Todo código, padrão ou afirmação de compatibilidade carrega
   ".NET Framework 4.5 / C# 5.0". Nunca afirme que código desta skill roda no .NET 8
   (ou .NET Core/5+) sem listar as adaptações.
2. **ADO.NET é factory genérica.** Use `DbProviderFactory` + `DbConnection`/`DbCommand`/
   `DbDataReader` + classe base `Dao` + `DaoFactory`. Não crie `SqlConnection`/
   `SqlCommand` direto (a teoria cita as classes concretas, o código nunca as usa, e o padrão as desaconselha).
   Não sugira EF, Dapper ou outro ORM como primeira opção — esta skill não usa ORM.
3. **Nunca reproduza BinaryFormatter.** O exemplo binário (serialização com
   `BinaryFormatter`) não faz parte desta skill: `BinaryFormatter` foi removido no .NET 9 e é inseguro por design. Para
   serializar, use `XmlSerializer` (o padrão desta skill) ou JSON moderno.
4. **LINQ é query syntax.** `from...where...orderby...select`. Method syntax
   (`.Where().OrderBy()`) só aparece como explicação do que o compilador faz —
   não é o idioma desta skill.
5. **MVVM é o clássico.** Mini-framework próprio `ClassicDesktop.Mvvm` (`Bindable` +
   `Command`), `DataContext` declarativo no XAML, `ListBox` + `ObservableCollection`,
   validação via `INotifyDataErrorInfo`, `IWindowServices` para o que a View precisa
   fazer. Não invente `RelayCommand`, `DataGrid` ou framework MVVM externo.
6. **Código literal só dos arquivos-fonte `.cs`/`.xaml`.** Vale o arquivo-fonte
   quando houver divergência com notas ou documentação secundária.
7. **Capstone são três apps.** Notepad → CadCliente → Chat, nesta ordem. Não invente
   um projeto final único nem etapas que não existem.
8. **Sintaxe moderna não entra.** Nada de `?.`, `$""`, namespaces file-scoped,
   anotações anuláveis, records, pattern matching ou `IAsyncEnumerable` — tudo isso é
   posterior ao C# 5.0.

## Tabela rápida — qual referência usar

| Tarefa | Referência |
|---|---|
| Filtrar/ordenar/agrupar listas em memória | [linguagem-csharp.md](references/linguagem-csharp.md) — LINQ query syntax |
| Ler/gravar no SQL Server sem ORM | [ado-net.md](references/ado-net.md) — factory + `Dao` |
| Tela WPF com lista editável | [wpf-mvvm.md](references/wpf-mvvm.md) — `Bindable` + `Command` |
| Serializar para arquivo | [defasagem.md](references/defasagem.md) + XML em [capstone.md](references/capstone.md) |
| Roteiro do projeto final | [capstone.md](references/capstone.md) — 3 apps |
| "Isto ainda vale?" / "roda no .NET 8?" | [defasagem.md](references/defasagem.md) |

## Erros comuns (observados no baseline)

| Erro | Correto (nesta skill) |
|---|---|
| `new SqlConnection(...)` direto | `DbProviderFactory` + `Dao`/`DaoFactory` |
| Sugerir Dapper/EF como primeira opção | ADO.NET puro; esta skill não usa ORM |
| `.Where().OrderBy()` como idioma LINQ | Query syntax `from...where...orderby...select` |
| `RelayCommand` / `DataGrid` / `CallerMemberName` | `Command` + `Bindable.SetValue`, `ListBox`, sem `CallerMemberName` |
| `DataContext` no code-behind como padrão | `DataContext` declarativo no XAML |
| Citar ".NET Framework 4.8" ou "SQL Server 2014" | 4.5; SQL Server Express sem ano afirmado |
| Reproduzir o exemplo binário | Recusar; oferecer `XmlSerializer`/JSON |
| Inventar um projeto final único | Três apps: Notepad, CadCliente, Chat |
| Sintaxe C# 6+ no código (`?.`, `$""`, `namespace X;`) | C# 5.0 apenas |

## Racionalizações proibidas

Atalhos que os testes mostraram e a skill veta — cada um com o contador explícito.
Se a resposta usa o atalho, está errada mesmo que o resto esteja certo.

| Atalho | Contador |
|---|---|
| "`SqlConnection` direto é mais simples" | Esta skill ensina a factory genérica e desaconselha `new SqlConnection`; o código-fonte nunca usa as classes concretas |
| "Dapper/EF como alternativa leve depois do padrão" | Esta skill não usa ORM em lugar nenhum; o mapeamento é manual, coluna por coluna |
| "Method syntax é o LINQ moderno, tanto faz" | Esta skill ensina query syntax; method syntax só aparece como explicação do que o compilador faz |
| "`RelayCommand`/`DataGrid` são o MVVM padrão" | Esta skill usa `Command`/`Bindable` próprios e `ListBox`; `RelayCommand` e `DataGrid` não fazem parte dela |
| "`DataContext` no code-behind via cast" | `DataContext` declarativo no XAML com instância nomeada; o code-behind acessa o ViewModel pelo nome (`viewModel`), não por cast |
| "O exemplo binário só para estudar" | Nunca reproduzir; `BinaryFormatter` removido no .NET 9 e inseguro por design |
| "Um `.cs` compila no net8.0, então o projeto migra" | Compilar um arquivo não é migrar o projeto: o `.csproj` (alvo 4.5), o WPF e as referências não migram juntos |
| "SQL Server 2014" / ".NET Framework 4.8" | 4.5; SQL Server Express sem ano afirmado |
| "Projeto final único" | Três apps: Notepad → CadCliente → Chat |
