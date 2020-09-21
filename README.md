# Entity Framework to EF Core Porting Cheat Sheet

Overview of porting Entity Framework to EF Core based on commonly-used features and APIs.

---
## Table of Contents
* [Required Changes](#required-changes)
* [DbContext Changes](#dbcontext-changes)
* [DbModelBuilder Changes](#dbmodelbuilder-changes)
* [Data Access and Tracking Changes](#data-access-and-tracking-changes)
* [Missing Features](#missing-features)

---
<br/> 
<!----------- INITIAL CHANGES ----------->

## Required Changes

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Description</th>
		<th>EF Example</th>
		<th>EF Core Example</th>
 	</tr>
  <tr>
    <td class="description">
      <b>Entity Framework namespace</b>
      <br/>
      Replaced by EF Core namespace
    </td>
    <td class="ef-example">
      <pre lang="csharp">
using System.Data.Entity;
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
using Microsoft.EntityFrameworkCore;
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Config files</b>
      <br/>
      <code>web.config</code> and <code>app.config</code> replaced by <code>appsettings.json</code>
    </td>
    <td class="ef-example">
      <pre lang="xml">
&lt;configuration&gt;
  &lt;connectionStrings&gt;
    &lt;add name="myConnection"
        connectionString="server=localhost;database=mydatabase;" /&gt;
  &lt;/connectionStrings&gt;
&lt;/configuration&gt;
      </pre>
    </td>
    <td class="ef-core-example">
      <code>appsettings.json</code>
      <pre lang="json">
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=MyDatabase;Trusted_Connection=True;"
  },
}
      </pre>
      <code>startup.cs</code>
      <pre lang="csharp">
using Microsoft.Extensions.Configuration;

public void ConfigureServices(IServiceCollection services)
{
services.AddDbContext&lt;MyDbContext&gt;(options =>
options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
}
</pre>
</td>
  </tr>
  <tr>
    <td class="description">
      <b>Spatial data types</b>
      <br/>
      Requires Nuget package: <code>NetTopologySuite</code>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
using Microsoft.SqlServer.Types;
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
using NetTopologySuite.Geometries;
<br/>
optionsBuilder.UseSqlServer(connectionString, options => options.UseNetTopologySuite());
      </pre>
    </td>
  </tr>
<!-- TEMPLATE 
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
END TEMPLATE -->
</table>

---

<!----------- DB CONTEXT ----------->

## DbContext Changes

Required changes for the DbContext class

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Description</th>
		<th>EF Example</th>
		<th>EF Core Example</th>
 	</tr>
  <tr>
    <td class="description">
      <b>Connection strings</b> 
      <br/>
      No longer passed into the base constructor. Set the connection string with the <code>DbContextOptionsBuilder</code> class.
    </td>
    <td class="ef-example">
      <pre lang="csharp">
public MyIssueTrackingContext(string nameOrConnectionString) 
    : base(nameOrConnectionString) 
{
}
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
private readonly string _connectionString;

public MyIssueTrackingContext(string connectionString)
{
&nbsp;&nbsp;&nbsp;&nbsp;\_connectionString = connectionString;
}
<br/>
public override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
&nbsp;&nbsp;&nbsp;&nbsp;optionsBuilder.UseSqlServer(\_connectionString);
}

</pre>
</td>

  </tr>
  <tr>
    <td class="description">
      <b>Registering database providers</b>
      <br/> Provider registration must be done in code using the <code>DbContextOptionsBuilder</code> class.
    </td>
    <td class="ef-example">
(set in web.config/app.config or with <code>DbConfiguration.SetProviderServices()</code> method)
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
// .UseSqlServer() registers Sql Server as the provider.
//  Change according the database being used
// Ex: .UseNpgsql() registers Postgres (also requires the 
//  Npgsql.EntityFrameworkCore.PostgreSQL Nuget package)
public override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
&nbsp;&nbsp;&nbsp;&nbsp;optionsBuilder.UseSqlServer(_connectionString);
}
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Eager Loading</b>
      <br/>
      Eagerly loading related entities using a <code>.Select()</code> drill-down has been replaced by <code>.ThenInclude()</code>.
    </td>
    <td class="ef-example">
      <pre lang="csharp">
context.Repos.Where(r => r.Name == "PortEF6ToCore")
    .Include(r => r.CreatedBy)
    .Include(r => r.Issues.Select(i => i.Assignees))
    .Include(r => r.Issues.Select(i => i.Comments));
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
context.Repos.Where(r => r.Name == "PortEF6ToCore")
    .Include(r => r.CreatedBy)
    .Include(r => r.Issues).ThenInclude(i => i.Assignees))
    .Include(r => r.Issues).ThenInclude(i => i.Comments));
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Lazy Loading</b>
      <br/>
      Must be enabled using the <code>DbContextOptionsBuilder</code> class or with an <code>ILazyLoader</code> service.
    </td>
    <td class="ef-example">
      <pre lang="csharp">
dbContext.Configuration.LazyLoadingEnabled = true; 
<br/>
dbContext.Configuration.LazyLoadingEnabled = false; 
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
// lazy loading disabled by default
// lazy loading enabled via proxies
optionsBuilder.UseLazyLoadingProxies()
      </pre>
      <pre lang="csharp">
// lazy loading enabled via ILazyLoader service
// requires adding the nuget package Microsoft.EntityFrameworkCore.Abstractions
using Microsoft.EntityFrameworkCore.Infrastructure;
<br/>
public class Repo
{
    private readonly ILazyLoader _lazyLoader;
    <br/>
    public Repo()
    {
    }
    <br/>
    public Repo(ILazyLoader lazyLoader)
    {
        _lazyLoader = lazyLoader;
    }
    <br/>
    private List<Issue> _issues;
    public List<Issue> Issues
    {
        get => _lazyLoader.Load(this, ref _issues);
        set => _issues = value;
    }
}
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Enabling proxy creation</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
context.Configuration.ProxyCreationEnabled = true;
<br/>
context.Configuration.ProxyCreationEnabled = false;
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
// proxies are disabled by default
optionsBuilder.UseLazyLoadingProxies()
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Logging generated SQL</b>
      <br/>
      Logging method is set using the <code>DbContextOptionsBuilder</code> class.
    </td>
    <td class="ef-example">
      <pre lang="csharp">
dbContext.Database.Log = Console.WriteLine;
      </pre>
    </td>
    <td class="ef-core-example">
    <pre lang="csharp">
using Microsoft.Extensions.Logging namespace;
<br/>
optionsBuilder.UseLoggerFactory(LoggerFactory.Create(Console.WriteLine));
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>DbSet.Local property</b>
      <br/>
      Now returns a <code>LocalView&lt;TEntity&gt;</code> instead of an <code>ObservableCollection&lt;T&gt;</code>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
var local = dbContext.Repos.Local;
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
var local = dbContext.Repos.Local.ToObservableCollection();
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Database Creation/Deletion</b>
      <br/>
      Methods to create/delete a database have been renamed.
    </td>
    <td class="ef-example">
      <pre lang="csharp">
dbContext.Database.CreateIfNotExists();
dbContext.Database.Delete();
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
dbContext.Database.EnsureCreated();
dbContext.Database.EnsureDeleted();
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Return type of </b><code>DbSet&lt;T&gt;.Add()</code>
      <br/>
      Changed from <code>T</code> to <code>EntityEntry&lt;T&gt;</code>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
Repo added = dbContext.Repos.Add(repo);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
Repo added = dbContext.Repos.Add(repo).Entity;
      </pre>
    </td>
  </tr>

<!-- TEMPLATE
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
END TEMPLATE -->
</table>

---

<!----------- DB MODELBUILDER ----------->

## DbModelBuilder Changes

DbModelBuilder, often used in the <code>DbContext.OnModelCreating()</code> method, requires the following changes:

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Description</th>
		<th>EF Example</th>
		<th>EF Core Example</th>
 	</tr>
  <tr>
    <td class="description">
      Renamed <code>DbModelBuilder</code> class to <code>ModelBuilder</code>
      <br/><br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
public void OnModelCreating(DbModelBuilder modelBuilder)
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
public void OnModelCreating(ModelBuilder modelBuilder)
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Many:Required relationship configuration</b>
      <br/>
      <code>.WithRequired()</code> replaced by <code>.WithOne().IsRequired()</code>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
modelBuilder.Entity&lt;Issue&gt;()
    .HasMany(i => i.Comments)
    .WithRequired(c => c.Issue)
    .HasForeignKey(c => c.IssueId)
    .WillCascadeOnDelete(true);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
modelBuilder.Entity&lt;Issue&gt;()
    .HasMany(i => i.Comments)
    .WithOne(c => c.Issue)
    .IsRequired(true)
    .HasForeignKey(c => c.IssueId)
    .OnDelete(DeleteBehavior.Cascade)
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Delete behavior</b>
      <br/>
      <code>.WillCascadeOnDelete()</code> replaced by <code>.OnDelete()</code>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
.WillCascadeOnDelete(true);
.WillCascadeOnDelete(false);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
.OnDelete(DeleteBehavior.Cascade);
.OnDelete(DeleteBehavior.Restrict);
      </pre>
    </td>
  </tr>

<!-- TEMPLATE
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
END TEMPLATE -->

## </table>

<!-- DATA ACCESS AND TRACKING -->

## Data Access and Tracking Changes

Changes in how data is accessed and tracked

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Description</th>
		<th>EF Example</th>
		<th>EF Core Example</th>
 	</tr>
  <tr>
    <td class="description">
      Renamed <code>DbChangeTracker</code> to <code>ChangeTracker</code>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
private void DisplayTrackedEntities(DbChangeTracker changeTracker)
{
}
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
using Microsoft.EntityFrameworkCore.ChangeTracking;
<br/>
private void DisplayTrackedEntities(ChangeTracker changeTracker)
{
}
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      Replaced <code>DbContextTransaction</code> with <code>IDbContextTransaction</code>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
DbContextTransaction transaction = dbContext.Database.BeginTransaction();
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
using Microsoft.EntityFrameworkCore.Storage;
<br/>
IDbContextTransaction transaction = dbContext.Database.BeginTransaction();
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Raw SQL Queries (returning entities)</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
// Sql string
DbSet.SqlQuery("SELECT * FROM MyTable");
<br/>
// Parameterized query
DbSet.SqlQuery("SELECT * FROM MyTable WHERE id={0}", id);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
// Sql string
DbSet.FromSqlRaw("SELECT * FROM MyTable");
<br/>
// Parameterized query
DbSet.FromSqlRaw("SELECT * FROM MyTable WHERE id={0}", id);
<br/>
// Parameterized query with interpolated string
DbSet.FromSqlInterpolated($"SELECT * FROM MyTable WHERE id={id}");
      </pre>
</td>

  </tr>
  <tr>
    <td class="description">
      <b>Raw SQL Queries (commands)</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
// Sql string
dbContext.Database.ExecuteSqlCommand("sp_UpdateAll");
<br/>
// Parameterized query
dbContext.Database.ExecuteSqlCommand("sp_InsertId {0}", id);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
// Sql string
dbContext.Database.ExecuteSqlRaw("sp_UpdateAll");
<br/>
// Parameterized query
dbContext.Database.ExecuteSqlRaw("sp_InsertId {0}", id);
<br/>
// Parameterized query with interpolated string
dbContext.Database.ExecuteSqlInterpolated($"sp_InsertId {id}");
      </pre>
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Compiled queries</b>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
using System.Data.Objects;
<br/>
CompiledQuery.Compile&lt;TContext, TParam1, ..., TParamN, TResult&gt;(
    // LINQ query
);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
using Microsoft.EntityFrameworkCore;
<br/>
EF.CompileQuery&lt;TContext, TParam1, ..., TParamN, TResult&gt;(
    // LINQ query
);
      </pre>
    </td>
  </tr>

<!-- TEMPLATE
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
END TEMPLATE -->
</table>

---

<!----------- MISSING FEATURES ----------->

## Missing Features

These Entity Framework features are missing from EF Core and may or may not be implemented in the future. If your project uses any of these features, consider using EF v6.3+

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Name</th>
		<th>Description</th>
		<th>Implementation Plan</th>
 	</tr>
  <tr>
    <td class="description">
      <b>TPC (Table-per-Concrete)</b>
    </td>
    <td class="ef-example">
      Allows entity classes with an abstract base class to be mapped to separate database tables (abstract base class remains unmapped)
    </td>
    <td class="ef-core-example">
      On backlog
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Entity splitting</b>
      <br/>
    </td>
    <td class="ef-example">
Allows single entities to be mapped to multiple tables (i.e. a subset of properties mapped to one table, another subset mapped to another table).
    </td>
    <td class="ef-core-example">
On backlog
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Visual Designer</b>
      <br/>
    </td>
    <td class="ef-example">
GUI-based db modeling tool that automatically generates DbContext and entity classes from a UML diagram. It also includes additional features such as auto-generated CUD stored procedures.
    </td>
    <td class="ef-core-example">
Not planned
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>ObjectContext class</b>
      <br/>
    </td>
    <td class="ef-example">
A low level class used to facilitate data access in a generic manner. It is commonly used when a stored procedure returns multiple result sets that each map to a different entity type.
    </td>
    <td class="ef-core-example">
Not planned
    </td>
  </tr>

<!-- TEMPLATE
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
END TEMPLATE -->
</table>

<!-- TABLE TEMPLATE
## Table title

Table description

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Description</th>
		<th>EF Example</th>
		<th>EF Core Example</th>
 	</tr>
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-example">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
</table>
END TABLE TEMPLATE -->
