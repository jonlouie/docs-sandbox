<<<<<<< HEAD
# Entity Framework to EF Core Porting Reference
=======
# Entity Framework to EF Core Porting Reference Guide
>>>>>>> 87a0b43... updates

A general reference for developers looking to migrate from Entity Framework to EF Core

---

## Table of Contents

- Syntactical Differences
  - [Namespace Changes](#namespace-changes)
  - [Configuration Changes](#configuration-changes)
  - [DbContext Changes](#dbcontext-changes)
  - [DbModelBuilder Changes](#dbmodelbuilder-changes)
  - [Data Access and Tracking Changes](#data-access-and-tracking-changes)
- [Behavioral Differences](#behavioral-changes)
- [Missing Features](#missing-features)

---

<br/> 
<!----------- INITIAL CHANGES ----------->

## Syntactical Differences

### Namespace Changes

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

### Configuration Changes

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
&nbsp;&nbsp;&nbsp;&nbsp;services.AddDbContext&lt;MyDbContext&gt;(options =>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
}

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

### DbContext Changes

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
public MyBloggingContext(string nameOrConnectionString) 
    : base(nameOrConnectionString) 
{
}
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
private readonly string _connectionString;

public MyBloggingContext(string connectionString)
{
&nbsp;&nbsp;&nbsp;&nbsp;\_connectionString = connectionString;
}
<br/>
public override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
&nbsp;&nbsp;&nbsp;&nbsp;optionsBuilder.UseSqlServer(\_connectionString);
}

</pre>
<hr/>
See "Configuration Changes" section above for an ASP.NET Core example with connection string configuration in a <code>startup.cs</code> file.
</td>

  </tr>
  <tr>
    <td class="description">
      <b>Registering database providers</b>
      <br/> Provider registration must be done in code using the <code>DbContextOptionsBuilder</code> class or in <code>startup.cs</code> 
    </td>
    <td class="ef-example">
(set in web.config/app.config or with <code>DbConfiguration.SetProviderServices()</code> method)
    </td>
    <td class="ef-core-example">
      <code>MyDbContext.cs</code>
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
<hr/>
See "Configuration Changes" section above for an ASP.NET Core example with database provider registration in a <code>startup.cs</code> file.
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
context.Blogs.Where(b => b.Title == "MyBlogTitle")
    .Include(b => b.Author)
    .Include(b => b.Posts.Select(p => p.Content))
    .Include(b => b.Posts.Select(p => p.Comments));
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
context.Blogs.Where(b => b.Name == "PortEF6ToCore")
    .Include(b => b.CreatedBy)
    .Include(b => b.Posts).ThenInclude(p => p.Content))
    .Include(b => b.Posts).ThenInclude(p => p.Comments));
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
// requires adding the Nuget package Microsoft.EntityFrameworkCore.Abstractions
using Microsoft.EntityFrameworkCore.Infrastructure;
<br/>
public class Blog
{
    private readonly ILazyLoader _lazyLoader;
    <br/>
    public Blog()
    {
    }
    <br/>
    public Blog(ILazyLoader lazyLoader)
    {
        _lazyLoader = lazyLoader;
    }
    <br/>
    private List<Post> _posts;
    public List<Post> Posts
    {
        get => _lazyLoader.Load(this, ref _posts);
        set => _posts = value;
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
var local = dbContext.Blogs.Local;
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
var local = dbContext.Blogs.Local.ToObservableCollection();
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
Blog added = dbContext.Blogs.Add(blog);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
Blog added = dbContext.Blogs.Add(blog).Entity;
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

### DbModelBuilder Changes

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
modelBuilder.Entity&lt;Blog&gt;()
    .HasMany(b => b.Posts)
    .WithRequired(p => p.Blog)
    .HasForeignKey(p => p.BlogId)
    .WillCascadeOnDelete(true);
      </pre>
    </td>
    <td class="ef-core-example">
      <pre lang="csharp">
modelBuilder.Entity&lt;Blog&gt;()
    .HasMany(b => b.Posts)
    .WithOne(p => p.Blog)
    .IsRequired(true)
    .HasForeignKey(p => p.BlogId)
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

</table>

<!-- DATA ACCESS AND TRACKING -->

### Data Access and Tracking Changes

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

<!----------- BEHAVIORAL CHANGES ----------->

## Behavioral Changes

These Entity Framework features exhibit different behavior in Entity Framework Core. Here's what to watch out for:

<table>
  <colgroup>
    <col style="width:20%">
    <col style="width:40%">
    <col style="width:40%">
  </colgroup>
	<tr>
		<th>Feature</th>
		<th>EF Behavior</th>
		<th>EF Core Behavior</th>
 	</tr>
  <tr>
    <td class="description">
      <b>Lazy loading</b>
    </td>
    <td class="ef-behavior">
Enabled by default
    </td>
    <td class="ef-core-behavior">
Not enabled by default
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Lazy loading by proxies</b>
    </td>
    <td class="ef-behavior">
Use of lazy loading proxies can be set on the fly with the follow statements:
      <pre lang="csharp">
context.Configuration.ProxyCreationEnabled = true;
context.Configuration.ProxyCreationEnabled = false;
      </pre>
    </td>
    <td class="ef-core-behavior">
Dynamically enabling/disabling lazy loading proxies no longer exists in EF Core. Use of lazy loading proxies can only be set upon DbContext instantiation.
    </td>
  </tr>
  <tr>
    <td class="description">
      <b>Change tracking</b>
    </td>
    <td class="ef-behavior">
[placeholder]
    </td>
    <td class="ef-core-behavior">
[placeholder]
    </td>
  </tr>

<!-- TEMPLATE
  <tr>
    <td class="description">
      <b>s</b>
      <br/>
    </td>
    <td class="ef-behavior">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="ef-core-behavior">
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
		<th>col1</th>
		<th>col2</th>
		<th>col3</th>
 	</tr>
  <tr>
    <td class="col1">
      <b>s</b>
      <br/>
    </td>
    <td class="col2">
      <pre lang="csharp">
s
      </pre>
    </td>
    <td class="col3">
      <pre lang="csharp">
s
      </pre>
    </td>
  </tr>
</table>
END TABLE TEMPLATE -->
