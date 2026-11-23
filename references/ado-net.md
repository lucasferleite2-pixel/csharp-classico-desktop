# ADO.NET (.NET Framework 4.5 / C# 5.0, SQL Server Express local)

Acesso a dados **sem ORM** no estilo clássico. Tese central:
**ADO.NET genérico (provider-independent) via `DbProviderFactory`, não `SqlClient`
direto.** As classes concretas `SqlConnection`/`SqlCommand` são citadas só na teoria
e nunca aparecem em nenhum `.cs`; o padrão desaconselha criar
`new SqlConnection` direto (o código ficaria atrelado ao SQL Server).

## Configuração — App.config literal

Literal de `Db/App.config`
(idêntico em `DAOFramework/TestDAO/App.config`):

```xml
<appSettings>
  <add key="dbProvider" value="System.Data.SqlClient" />
</appSettings>

<connectionStrings>
  <add name="dbConnString" connectionString="Data Source=(local)\SQLEXPRESS;Initial Catalog=testdb;Integrated Security=True" />
</connectionStrings>
```

Forma: `Data Source=(local)\SQLEXPRESS` (instância local) + `Initial Catalog=testdb` +
`Integrated Security=True` (autenticação Windows, sem usuário/senha). O `App.config`
também fixa `<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />`.
**As chaves corretas são `dbProvider` e `dbConnString`**.
Esta skill ensina ainda `SqlConnectionStringBuilder` como alternativa a escrever a
string à mão. `ConfigurationManager` exige referenciar manualmente `System.Configuration.dll`.

## Factory + leitura — literal de `Db/Program.cs`

```csharp
static string connString;
static DbProviderFactory factory;

static void Main()
{
    factory = DbProviderFactories.GetFactory(ConfigurationManager.AppSettings["dbProvider"]);
    connString = ConfigurationManager.ConnectionStrings["dbConnString"].ConnectionString;

    using (DbConnection conn = factory.CreateConnection())
    {
        conn.ConnectionString = connString;
        conn.Open();
```

Leitura com `DbDataReader` (por nome de coluna — por índice também é possível, mas
por nome é mais intuitivo):

```csharp
using (DbCommand cmd = factory.CreateCommand())
{
    cmd.Connection = conn;
    cmd.CommandText = "SELECT id, nome, valor, descricao FROM produto ORDER BY id";

    using (DbDataReader dr = cmd.ExecuteReader())
    {
        while (dr.Read())
        {
            Produto p = new Produto();
            p.Id = (int)dr["id"];
            p.Nome = (string)dr["nome"];
            p.Valor = Convert.ToDouble((decimal)dr["valor"]);
            p.Descricao = (string)dr["descricao"];
            produtos.Add(p);
        }
    }
}
```

`Find` usa o mesmo padrão com `if (!dr.Read()) return null;`.

## Os três métodos de execução

| Método | Uso | Retorno |
|---|---|---|
| `ExecuteNonQuery()` | INSERT / UPDATE / DELETE | `int` — nº de registros afetados |
| `ExecuteReader()` | SELECT | `DbDataReader` — iterar com `while (dr.Read())` |
| `ExecuteScalar()` | SELECT de valor único (`SELECT COUNT(*)`, `SELECT MAX(idade)`) | `object` — 1º valor da 1ª linha/coluna |

```csharp
cmd.CommandText = "SELECT COUNT(*) FROM produto";
int count = (int)cmd.ExecuteScalar();
```

## Comandos parametrizados — literal de `Db/Program.cs`

```csharp
DbCommand cmd = factory.CreateCommand();
cmd.Connection = conn;
cmd.CommandText = "INSERT INTO produto (nome, valor, descricao) VALUES (@Nome, @Valor, @Descricao)";

DbParameter param = factory.CreateParameter();
param.ParameterName = "@Nome";
param.Value = p.Nome;
cmd.Parameters.Add(param);
// ... repete para @Valor e @Descricao
cmd.ExecuteNonQuery();
```

Placeholders `@Nome`/`@Valor` (sintaxe SQL Server). Justificativa: sem parâmetro
a query é remontada e re-parseada a cada chamada; com parâmetro, parse uma vez e só os
valores são substituídos. A concatenação é o anti-padrão (o argumento dado é performance). Tipo do parâmetro: mapeamento automático
C#→banco; pode-se fixar via `param.DbType` (enum genérico) ou `SqlDbType` (amarra ao
SQL Server).

**Nulos (só teoria):** `Convert.IsDBNull(obj)` ou comparar com `DBNull.Value`; **nunca**
comparar com o `null` do C# — o NULL do banco é outro valor.

## Estrutura genérica — `Dao` + `DaoFactory`

Solução `DAOFramework.sln`: biblioteca **DAO** (namespace `Db`, vira DLL reutilizável) +
console **TestDAO** (só testes). Classe base abstrata literal de `DAOFramework/DAO/Dao.cs`:

```csharp
namespace Db
{
    public abstract class Dao
    {
        static DbProviderFactory providerFactory;
        static string connStr;
        static bool initialized;

        public static void Init(string providerName, string connString)
        {
            providerFactory = DbProviderFactories.GetFactory(providerName);
            connStr = connString;
            initialized = true;
        }

        public static void Init()
        {
            Init(ConfigurationManager.AppSettings["dbProvider"], ConfigurationManager.ConnectionStrings["dbConnString"].ConnectionString);
        }

        protected DbConnection CreateConnection()
        {
            if (!initialized)
            {
                Init();
            }

            DbConnection conn = providerFactory.CreateConnection();
            conn.ConnectionString = connStr;
            conn.Open();
            return conn;
        }

        protected DbCommand CreateCommand(DbConnection conn, string text)
        {
            DbCommand cmd = providerFactory.CreateCommand();
            cmd.Connection = conn;
            cmd.CommandText = text;
            return cmd;
        }

        protected DbParameter CreateParameter(DbCommand cmd, string name, object value, DbType? type)
        {
            DbParameter param = providerFactory.CreateParameter();
            param.ParameterName = name;
            param.Value = value;

            if (type.HasValue)
            {
                param.DbType = type.Value;
            }

            cmd.Parameters.Add(param);
            return param;
        }
    }
}
```

Pontos: `abstract` + `public`; estado **estático e compartilhado**; `Init()` com 2
sobrecargas (explícita ou por convenção lendo o `App.config`); inicialização preguiçosa
(`CreateConnection()` chama `Init()` sozinho); `CreateConnection()` já devolve a conexão
**aberta**; `CreateParameter` recebe `DbType?` (nullable — `null` = deixar o ADO.NET
inferir o tipo, que é o que o exemplo faz em todas as chamadas).

Literal de `DAOFramework/DAO/DaoFactory.cs`:

```csharp
public static class DaoFactory
{
    public static T CreateDao<T>() where T : Dao, new()
    {
        return new T();
    }
}
```

Regra: a aplicação nunca dá `new` direto num DAO; centralizar a criação
para poder mudar o comportamento num só lugar.

DAO concreto + uso — literal de `DAOFramework/TestDAO/Program.cs`:

```csharp
ProdutoDAO dao = DaoFactory.CreateDao<ProdutoDAO>();

Produto p = new Produto();
p.Nome = "Televisão";
p.Valor = 1600;
p.Descricao = "Televisão de LED";

dao.Inserir(p);

p = dao.Carregar(12);
Console.WriteLine(p);
```

```csharp
public void Inserir(Produto p)
{
    using (DbConnection conn = CreateConnection())
    {
        using (DbCommand cmd = CreateCommand(conn, "INSERT INTO produto (nome, descricao, valor) VALUES (@Nome, @Descricao, @Valor)"))
        {
            CreateParameter(cmd, "@Nome", p.Nome, null);
            CreateParameter(cmd, "@Descricao", p.Descricao, null);
            CreateParameter(cmd, "@Valor", p.Valor, null);
            cmd.ExecuteNonQuery();
        }
    }
}

public Produto Carregar(int id)
{
    using (DbConnection conn = CreateConnection())
    {
        using (DbCommand cmd = CreateCommand(conn, "SELECT id, nome, descricao, valor FROM produto WHERE id = @Id"))
        {
            CreateParameter(cmd, "@Id", id, null);
            using (DbDataReader dr = cmd.ExecuteReader())
            {
                if (!dr.Read())
                {
                    return null;
                }

                return new Produto()
                {
                    Id = (int)dr["id"],
                    Nome = (string)dr["nome"],
                    Descricao = (string)dr["descricao"],
                    Valor = Convert.ToDouble((decimal)dr["valor"])
                };
            }
        }
    }
}
```

Regra de organização: **1 DAO por tabela** (ou 1 por conceito); a camada DAO é a **única**
que fala com o banco — a aplicação só manipula a entidade e chama o DAO. Entidade de
exemplo: `Produto { int Id; string Nome; double Valor; string Descricao; }`, tabela
`produto` no banco `testdb`. (Só `Inserir` e `Carregar` estão implementados neste exemplo;
Update/Delete/List/Count não existem aqui.)

## Regras de conexão

Conexão é "recurso valioso e escasso" — **sempre fechar, mesmo com exceção**: bloco
`using` (chama `Dispose()` ao fim — usado para `DbConnection`, `DbCommand` e
`DbDataReader`) ou `try/finally` com `Close()` no `finally`. Conexões órfãs esgotam o
pool e travam a aplicação. Em `Db/Program.cs`, `Main` abre **uma** conexão e a reutiliza
para todos os métodos; em `ProdutoDAO`, cada método abre e fecha a sua.

Assimetria real no código (vale o `.cs`): em `Db/Program.cs`, `Insert`/`Update`/`Delete`
criam o `DbCommand` **sem** `using` nem `Dispose`; no exemplo `DAOFramework` o padrão é uniforme, com
`using` aninhados. Prefira o padrão uniforme.

## O que esta skill NÃO cobre aqui

- **Nenhum ORM** — EF, Dapper, LINQ to SQL: o mapeamento é manual
  (`dr["coluna"]` → propriedades, com casts).
- **Nenhum `DataAdapter`/`DataSet`/`DataTable`** — só o modo conectado com
  `DbDataReader`. Não há `Fill()` nem modo desconectado.
- **Transações sem código** — só a teoria (`conn.BeginTransaction()` retorna
  `DbTransaction`; `Commit()` efetiva, `Rollback()` desfaz). Nenhum exemplo executa
  transação aqui (o projeto CadCliente usa — ver [capstone.md](capstone.md)).
- **Nada assíncrono** — só métodos síncronos. Nada de pool, retry, migração de schema
  ou DDL (a tabela já existe; aqui só DML/SELECT).
