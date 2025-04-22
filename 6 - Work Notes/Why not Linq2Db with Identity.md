the consuming application will be responsible for creating the actual database tables. However, ASP.NET Core Identity is tightly integrated with Entity Framework Core (EF Core), not Linq2DB.

Here's why we should use EF Core for this library:

1. Built-in Integration: Identity provides ready-to-use EF Core components (IdentityDbContext, UserStore, RoleStore) that handle all the data persistence logic for users, roles, claims, etc. Replicating this functionality for Linq2DB would mean writing custom Identity stores from scratch, which is complex and defeats the purpose of using the standard Microsoft solution.

2. Schema Definition: The ApplicationDbContext we create in Aturis.AuthenticationNew will define the schema for the Identity tables (including your custom TenantId and Status fields on the ApplicationUser).

3. Migrations: The consuming application will use EF Core Migrations, pointing to the ApplicationDbContext defined in our library, to create or update its MySQL database schema accordingly.

So, while you use Linq2DB elsewhere, for Aturis.AuthenticationNew to leverage ASP.NET Core Identity effectively, we need to use EF Core. We'll use the Pomelo.EntityFrameworkCore.MySql provider package since you're using MySQL. The library itself won't contain MySQL-specific code, but it will define the EF Core model, and the consuming app will configure EF Core to use the MySQL provider.