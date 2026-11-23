# C# clássico desktop

Skill para escrever código desktop .NET clássico: fundamentos da linguagem
C# 5.0, acesso a dados com ADO.NET sem ORM, WPF com MVVM, e três aplicações
finais. Tudo é **.NET Framework 4.5 / C# 5.0 / Visual Studio 2013 / SQL
Server Express local** — a skill marca a versão em cada bloco e nunca
apresenta nada disto como .NET atual.

## O que cobre

- **Linguagem:** fundamentos do C# 5.0, coleções, LINQ em query syntax
  (`from...where...orderby...select`), LINQ to XML, delegates, lambdas,
  tipos anônimos e `var`.
- **Dados:** ADO.NET genérico via `DbProviderFactory`, sem ORM — classe base
  `Dao` + `DaoFactory`, comandos parametrizados, `DbDataReader`, os três
  métodos de execução (`ExecuteNonQuery` / `ExecuteReader` /
  `ExecuteScalar`), `DBNull`, regra de sempre fechar a conexão.
- **Interface:** WPF + MVVM com mini-framework próprio (`Bindable` +
  `Command`, namespace `ClassicDesktop.Mvvm`), `DataContext` declarativo no
  XAML, `ListBox` + `ObservableCollection`, validação via
  `INotifyDataErrorInfo`, `IWindowServices` para o que a View precisa fazer.
- **Capstone:** três aplicações em ordem crescente — Notepad (WPF+MVVM),
  CadCliente (CRUD + ADO.NET/DAO + SQL Server, com transações) e Chat
  (multiusuário TCP, porta 3000, protocolo texto de 11 tipos).
- **Serialização:** `XmlSerializer` com exemplo literal completo
  (serializar/desserializar, `[XmlRoot]` / `[XmlElement]` / `[XmlAttribute]`).

## O que NÃO cobre

Nada de Entity Framework (nem como alternativa ao ADO.NET desta skill),
nada de `BinaryFormatter` (removido no .NET 9 — a skill veta reproduzir o
exemplo binário, nem como "opção legada"), nada de sintaxe posterior ao
C# 5.0 (`?.`, `$""`, namespaces file-scoped, anotações anuláveis, records,
pattern matching, `IAsyncEnumerable`), nada de `DataAdapter` / `DataSet` /
`DataTable` (só o modo conectado com `DbDataReader`), nada assíncrono e
nada de instalação do SQL Server (a skill assume o banco local já
instalado). Transações aparecem só na teoria em ADO.NET e no projeto
CadCliente.

## Avisos de datação (regras da skill)

- **Tudo aqui é de 2013.** .NET Framework 4.5, C# 5.0, Visual Studio 2013
  (artefatos `.v12.suo`, `ToolsVersion 12.0`), SQL Server Express local
  `(local)\SQLEXPRESS` sem ano afirmado.
- **Código da época não é código atual.** Todo bloco carrega o carimbo de
  versão; a skill nunca afirma que o projeto roda no .NET 8 sem listar as
  adaptações. O exemplo de serialização XML compila num projeto net8.0
  (0 erros, 5 warnings); o WPF do .NET Framework, `System.Configuration`
  antigo, `AppDomain`, Remoting, WCF, `System.Web`, EF6 e
  `BinaryFormatter` não migram direto.
- **`BinaryFormatter` foi removido no .NET 9** e é inseguro por design.
  Para serializar: `XmlSerializer` ou JSON moderno.
- **Legado mantido como tal:** `ArrayList` + `OfType<T>()`, thread por
  cliente no chat e `System.Data.SqlClient` via `DbProviderFactory` são
  arquitetura da época, não prática corrente.

## Como usar

Aponte o agente para `SKILL.md`. A skill define o fluxo, as regras duras
e as racionalizações proibidas; os detalhes vivem em `references/`:

| Arquivo | Conteúdo |
|---|---|
| `SKILL.md` | Fluxo, tabela rápida de decisão, regras duras, erros comuns |
| `references/linguagem-csharp.md` | Fundamentos do C# 5.0 + LINQ query syntax + LINQ to XML |
| `references/ado-net.md` | ADO.NET genérico: factory, `Dao`/`DaoFactory`, parâmetros |
| `references/wpf-mvvm.md` | WPF + MVVM: `Bindable`, `Command`, bindings, validação |
| `references/capstone.md` | As 3 aplicações finais + exemplo literal de XML |
| `references/defasagem.md` | O que é datado, o que não existe no C# 5.0, o que está excluído |

## Método

Construída pelo método Superpowers `writing-skills` (TDD aplicado a
skills): 6 cenários de teste (baseline 0/6 → com skill 6/6) + 2
micro-testes de pressão (T1/T2, ambos PASS).

## Licença

MIT — ver `LICENSE`.
