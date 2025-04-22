Tags: #wartungsapps

Source: https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/

# New DbContext instances
The fastest way to create a new DbContext instance is by using `new` to create a new instance. However, there are scenarios that require resolving additional dependencies:
- Using [[#DbContext Lifetime, Configuration, and Initialization]] to configure the context.
- Using a connection string per DbContext, such as when you use [ASP.NET Core's Identity model](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/customize-identity-model?view=aspnetcore-9.0). For more information, see [Multi-tenancy (EF Core documentation)](https://learn.microsoft.com/en-us/ef/core/miscellaneous/multitenancy).
# DbContext Lifetime, Configuration, and Initialization
