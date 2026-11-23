# Linguagem C# (.NET Framework 4.5 / C# 5.0 / VS 2013)

Fundamentos da linguagem no estilo clássico. Tudo aqui é C# 5.0 — ver
[defasagem.md](defasagem.md) para o que NÃO existe nesta versão.

## LINQ — query syntax é o idioma desta skill

LINQ aqui é quase 100% em **query syntax**. Method syntax aparece só como explicação
do que o compilador faz por baixo dos panos. Nenhum exemplo usa `.Where(`, `.OrderBy(`,
`.Select(` ou `.GroupBy(`.

Literal de `LINQBasico/Program.cs`:

```csharp
var q = from a in alunos
        where a.Nota >= 5 && a.Nome.StartsWith("J")
        orderby a.Nota ascending
        select new { Inicial = a.Nome[0], Nota = a.Nota * 10 };
```

Literal de `LINQAvancado/Program.cs` (agrupar e left join):

```csharp
var q = from a in alunos
        from e in a.Atividades
        group e by e.Nome into g
        select new { Atividade = g.Key, Qtde = g.Count() };

var q = from a in alunos
        join f in familias on a.Nome equals f.Filho into af
        from f in af.DefaultIfEmpty()
        select new { Pai = f == null ? "-" : f.Pai, Nota = a.Nota };

var q = from a in alunos
        where a.Turma.Serie == 3 && a.Turma.Letra == 'A'
        select a.Nota;
```

### Cláusulas de query syntax ensinadas

| Cláusula | Uso |
|---|---|
| `from` (incl. segundo `from` sobre coleção aninhada) | Fonte da query; `from e in a.Atividades` |
| `where` | Filtro |
| `orderby ... ascending/descending` | Ordenação (`ascending` é o default) |
| `select` | Projeção: objeto inteiro, uma property, ou tipo anônimo |
| `group ... by ... into` | Agrupamento; `g.Key`, `g.Count()` |
| `join ... on ... equals` | Inner join |
| `join ... into ...` + `from ... in af.DefaultIfEmpty()` | Left join, com checagem manual de `null` na projeção |
| `let` | Variável intermediária dentro da query |

### Só como métodos de extensão (sem equivalente em query syntax)

- Agregações: `Count`, `Min`, `Max`, `Average`, `Sum`
- Conjuntos: `Union`, `Intersect`, `Except`, `Concat`, `Distinct`
- Conversão/execução imediata: `ToList()`, `ToArray()`, `ToDictionary(chave => ...)`
- `OfType<T>()` para coleções não genéricas (`ArrayList`)
- Geração: `Enumerable.Range(início, qtde)`, `Enumerable.Empty<T>()`, `Enumerable.Repeat(elemento, qtde)`

Literal de `LINQGeracao/Program.cs`:

```csharp
var c = Enumerable.Repeat("B", 10);
// variantes comentadas: Enumerable.Range(1, 10); Enumerable.Empty<double>();
```

### Conceitos

- **Execução adiada vs. imediata:** a query só executa na iteração ou ao chamar
  `ToList()`/`ToArray()`/`ToDictionary()`.
- **`var` + tipos anônimos:** `select new { Inicial = ..., Nota = ... }`; `var` é
  inferência em tempo de compilação, só para variável local — proibido como retorno
  de método. O compilador gera a classe e sobrescreve `ToString()`.
- **Métodos de extensão:** LINQ funciona porque `System.Linq.Enumerable` adiciona
  extension methods a `IEnumerable<T>`; sem `using System.Linq;` nada compila.
- **Estilo de exemplo:** massa de dados em memória montada à mão (`CriarAlunos()`/
  `LerItens()`), `List<T>` + object initializer. Domínios: escola (Aluno, Nota, Turma,
  Atividades) e supermercado no exercício (agrupar por Tipo, Min/Max/Média de preço).

### LINQ to XML

Mesma query syntax sobre `XElement`/`XDocument`:

```csharp
// padrão clássico: from e in xml.Elements("colaborador"), e.Attribute("...")
```

Criação de XML por construção funcional com `new XElement(...)`.

## Fundamentos

- **Coleções:** `List<T>` como fonte padrão de LINQ to Objects; `Dictionary` via
  `ToDictionary(a => a.Nome)`; `ArrayList` (não genérica) só funciona com `OfType<T>()`.
- **Lambda:** parâmetro único sem parênteses; dois ou mais exigem
  parênteses; tipo do parâmetro opcional; closures. Lambdas aparecem no LINQ como
  seletor de chave (`g.Min(i => i.Preco)`) e em `List<T>.ForEach(...)`.
- **Delegates:** delegates nomeados, callback, genéricos `Action`/`Func` —
  base para entender que os métodos LINQ recebem delegates.
- **Tipos anônimos + `var`:** ver acima.
- **Operadores e casting:** `checked`/`unchecked`, sobrecarga de operadores,
  indexadores, castings customizados.

## O que NÃO existe no C# 5.0 (não use)

- Null-conditional `?.` — esta skill faz checagem manual de null em todo lugar
  (`f == null ? "-" : f.Pai`; hoje seria `f?.Pai ?? "-"`).
- Interpolação de string `$""` — aqui usa-se `string.Format("{0} -> {1}", ...)` e
  concatenação.
- Records, pattern matching, `IAsyncEnumerable`, `Chunk`/`DistinctBy`/`MaxBy`
  (.NET 6+), tuplas como chave de `GroupBy`.
