![Release workflow](https://github.com/0UserName/testcontainers.ado/actions/workflows/release.yml/badge.svg)



# Motivation

<div align="justify">

The library simplifies the creation of integration tests that require database access. It is particularly useful for testing database-driver-specific functionality, such as driver plugins.

</div>



# Usage

<div align="justify">

Define a custom fixture that provides test runners and specify it using the `TestFixtureSource` attribute on the test class:

</div>



```csharp
internal static class PostgreSqlFixture
{
    public static IEnumerable Runners
    {
        get
        {
            yield return new TestFixtureData(new PostgreSqlRunner("postgres:14-alpine", "Init", null, (NpgsqlDataSourceBuilder builder) => builder.UseTvp())); // Can be combined with SqlServerRunner.
        }
    }
}
```



<div align="justify">

The test class should inherit from `AbstractDbTests<TDbRunner>`, using the runner type provided by the fixture as the generic type argument:

</div>



```csharp
[TestFixtureSource(typeof(PostgreSqlFixture), nameof(PostgreSqlFixture.Runners))]
public sealed class InsertTests<TDbRunner>(TDbRunner runner) : AbstractDbTests<TDbRunner>(runner) where TDbRunner : IDbRunner
{
    [TestCase("dbo.procedure1")]
    [TestCase("dbo.procedure2")]
    [TestCase("dbo.procedure3")]
    public async Task TestDataTableAsync(string procedure)
    {
        await Runner.ThatOutputRowsAsync(procedure, DbParameterNames.O_P, table.Rows.Count, table.AsIn(default), default(long).AsOut(default)); // Initialize the table variable beforehand.
    }
}
```



<div align="justify">

The `Runner` property, defined in the base class, provides access to the runner, which exposes `IDbCommand` and [DbDataSource](https://learn.microsoft.com/en-us/dotnet/api/system.data.common.dbdatasource?view=net-10.0). `IDbCommand` currently provides predefined methods for interacting with stored procedures only. These methods can be used to implement custom test logic. If they are insufficient, `DbDataSource` can be used to implement any required database-related functionality.

</div>



> [!NOTE]
> See additional examples in the following projects: [DbExtensions.Tvp](https://github.com/0UserName/DbExtensions.Tvp/tree/master/DbExtensions.Tvp.Tests.Integrations) and [Npgsql.Tvp](https://github.com/0UserName/Npgsql.Tvp/tree/master/Npgsql.Tvp.Tests).