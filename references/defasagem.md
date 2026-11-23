# Defasagem — o que é .NET 4.5 / C# 5.0 e o que não entra

Esta skill descreve código de **2013**: .NET Framework 4.5, C# 5.0, Visual Studio
2013, SQL Server Express local. Nada aqui é .NET atual. Regra: todo bloco da skill
carrega a versão; nunca apresente um padrão clássico como prática corrente sem o
carimbo de versão.

## O que é datado (e continua na skill, marcado)

| Item | Status clássico | Na skill |
|---|---|---|
| .NET Framework 4.5 | Runtime clássico | Carimbado em cada bloco |
| C# 5.0 | Linguagem clássica | Sintaxe da época apenas (ver abaixo) |
| VS 2013 (`.v12.suo`, `ToolsVersion 12.0`) | IDE clássica | Referência histórica |
| SQL Server Express `(local)\SQLEXPRESS` | Banco clássico | Sem ano afirmado |
| `System.Data.SqlClient` via `DbProviderFactory` | Padrão clássico | Ensinado como o padrão desta skill |
| `XmlSerializer` | Serializador clássico | Válido até hoje; ensinado com o exemplo literal |
| Mini-framework próprio `ClassicDesktop.Mvvm` | MVVM clássico | Ensinado como o padrão desta skill |
| `ArrayList` + `OfType<T>()` | Caso ensinado no LINQ | Legado hoje; mantido como ensinado aqui |
| Thread por cliente (chat) | Modelo do capstone | Arquitetura da época; não é o modelo moderno |

## O que NÃO existe no C# 5.0 — nunca escreva na skill nem no código

- Null-conditional `?.` e null-coalescing assignment — esta skill faz checagem manual
  de null (`f == null ? "-" : f.Pai`).
- Interpolação de string `$""` — aqui usa-se `string.Format("{0} -> {1}", ...)` e
  concatenação.
- Namespaces file-scoped (`namespace X;`), anotações anuláveis (`Produto?`,
  `string.Empty` como padrão), `CallerMemberName` como idioma corrente — tudo
  posterior a 2013. (O `Bindable` desta skill usa `[CallerMemberName]` como parâmetro
  opcional — isso existe desde o C# 5.0 via `System.Runtime.CompilerServices` e é
  literal do código-fonte; não é um acréscimo.)
- Records, pattern matching, `IAsyncEnumerable`, `Chunk`/`DistinctBy`/`MaxBy`
  (.NET 6+), `ToDictionary` com comparador customizado, `GroupBy` com chave composta
  via tupla.

## O que está EXCLUÍDO (não mencionado como opção)

- **Entity Framework 6 model-first/database-first:** fora do escopo desta skill
  (a teoria de ORM também está fora). Não ensine EF como alternativa ao ADO.NET desta skill.
- **BinaryFormatter:** removido no .NET 9, inseguro por design
  (desserialização instancia tipos arbitrários — RCE). **Nunca reproduza o exemplo
  binário, nem como "opção legada".** Para serializar: `XmlSerializer` (clássico) ou
  JSON moderno (`System.Text.Json`); para binário compacto e seguro: protobuf-net /
  MessagePack. Arquivos antigos em binário: migração única em ambiente controlado,
  não manutenção do formato.
- **Instalação do SQL Server:** fora do escopo; a skill assume o banco
  local já instalado.

## "Esse código roda no .NET 8?" — como responder

1. Diga a origem: **.NET Framework 4.5 / C# 5.0 / VS 2013**.
2. Não afirme compatibilidade direta. O que roda com pouco ajuste: console/biblioteca
   com só `System`, `Collections.Generic`, `Linq`, `IO`, `Xml.Serialization` (o
   exemplo `Serializacao-XML` compila num projeto net8.0 — 0 erros, ver teste abaixo).
3. O que **não** roda direto: WPF do .NET Framework (precisa migrar para WPF no .NET
   moderno ou WinUI), `System.Configuration` antigo, `AppDomain`, Remoting, WCF,
   `System.Web`, EF6 (o caminho moderno é EF Core), `BinaryFormatter` (removido).
4. Peça o `.csproj`/código quando a pergunta vier sem contexto — não adivinhe a origem.

## Teste de compilação

Amostra `Serializacao-XML/Program.cs` compilada num projeto console net8.0 com o
.NET 8 SDK: **0 erros, 5 warnings** (só do analisador moderno de referências
anuláveis, inexistente no C# 5.0). O `.csproj` original (alvo .NET Framework 4.5)
não foi construído — no Linux, precisaria do pacote de assemblies de referência.
