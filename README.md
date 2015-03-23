# SQLiteEfCodeFirstDbCreator
Creates a [SQLite Database](https://sqlite.org/) from Code, using [Entity Framework](https://msdn.microsoft.com/en-us/data/ef.aspx) CodeFirst.

This Project ships a "SqliteContextInitializer" which creates a new SQLite Database, based on your model/code.
I started with the code from flaub: https://gist.github.com/flaub/1968486e1b3f2b9fddaf

Currently the following is supported:
- Tables from classes
- Columns from properties
- ForeignKey constraint (1-n relationships)
- Cascade on delete
- PrimaryKey constraint (An int PrimaryKey will automatically be incremented)
- Not Null constraint

I tried to write the code in a extensible way.
The logic is devided into two main parts. Builder and Statement.
The Builder-Part knows how to translate the EdmModel into statements where a statement class creates the SQLite-SQL-Code. The structure of the statements is influenced by the [SQLite Language Specification](https://www.sqlite.org/lang.html).

## How to use
If you want to let the Entity Framework create the database, if it does not exist, just set `SqliteContextInitializer<>` as your `IDbInitializer`.
```csharp
public class MyDbContext : DbContext
{
    public TestDbContext()
        : base("ConnectionStringName") { }
  
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        var sqliteConnectionInitializer = new SqliteContextInitializer<TestDbContext>(
            Database.Connection.ConnectionString, modelBuilder);
        Database.SetInitializer(sqliteConnectionInitializer);
    }
}
```

In a more advanced szenario you may want to populate some core- or test-data after the database was created.
To do this, inherit from `SqliteContextInitializer<>`
```csharp
public class TestDbContextInitializer : SqliteContextInitializer<TestDbContext>
{
    public TestDbContextInitializer(string connectionString, DbModelBuilder modelBuilder)
        : base(connectionString, modelBuilder) { }

    protected override void Seed(TestDbContext context)
    {
        context.Set<Player>().Add(new Player());
    }
}
```
