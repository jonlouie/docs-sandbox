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
    <td class="col1">
      Replace Entity Framework namespace
      <br><br>
    </td>
    <td class="col2">
      <pre lang="csharp">
using System.Data.Entity;
      </pre>
    </td>
    <td class="col3">
      <pre lang="csharp">
using Microsoft.EntityFrameworkCore;
      </pre>
    </td>
  </tr>
  <tr>
    <td class="col1">
      <b><code>web.config</code> and <code>app.config</code> replaced by <code>appsettings.json</code></b>
    </td>
    <td class="col2">
      <pre lang="xml">
&lt;configuration&gt;
  &lt;connectionStrings&gt;
    &lt;add name="myConnection" connectionString="server=localhost;database=mydatabase;" /&gt;
  &lt;/connectionStrings&gt;
&lt;/configuration&gt;
      </pre>
    </td>
    <td class="col3">
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
services.AddDbContext<MyDbContext>(options =>
options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
}
      </pre>
    </td>
  </tr>
  <tr>
    <td class="col1">
      <b>Spatial data types</b>
      <br>
      Requires Nuget package: <code>NetTopologySuite</code>
    </td>
    <td class="col2">
      <pre lang="csharp">
using Microsoft.SqlServer.Types;
      </pre>
    </td>
    <td class="col3">
      <pre lang="csharp">
using NetTopologySuite.Geometries;
<br>
optionsBuilder.UseSqlServer(connectionString, options => options.UseNetTopologySuite());
      </pre>
    </td>
  </tr>
</table>
