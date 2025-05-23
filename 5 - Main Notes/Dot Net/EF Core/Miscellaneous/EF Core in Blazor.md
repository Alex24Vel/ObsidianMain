Tag: 
Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/blazor-ef-core

# Database access

EF Core relies on a [[DbContext]] as the means to configure database access and act as a [[Unit Of Work]].

EF Core provides the AddDbContext extension for ASP.NET Core apps registers the context as a **scoped** service by default.

[[DbContext]] isn't thread safe and isn't designed for concurrent use. The existing lifetimes are inappropriate for these reasons:

- **Singleton** shares state across all users of the app and leads to inappropriate concurrent use.
- **Scoped** (the default) poses a similar issue between components for the same user.
- **Transient** results in a new instance per request; but as components can be long-lived, this results in a longer-lived context than may be intended.

## Mitigate errors:

- **Use one context per operation. The context is designed for fast, low overhead instantiation:**
```C#
using var context = new ProductsDatabaseContext();
return await context.Products.ToListAsync();
```
- **Use a flag to prevent multiple concurrent operations:**

```C#
if (Loading)
    return;
try {
    Loading = true;
    //... DATABASE OPERATIONS GO HERE==
}
finally {
    Loading = false;
}
```
The loading logic is used to disable UI controls so that users don't inadvertently select buttons or update fields while data is fetched.
- If there's any chance that multiple threads may access the same code block, inject a [[DBContextFactory]] and make a new instance per operation. Otherwise, injecting and using the context is usually sufficient.
- For longer-lived operations that take advantage of EF Core's [[Change Tracking]] or [[Concurrency control]], [[##Scope a database context to the lifetime of the component]]

# New DbContext instances
**See**: [[DbContext]]

## Scope a database context to a component method
The factory is injected into the component:
`@inject IDbContextFactory<ContactContext> DbFactory`

Create a [[DbContext]] for a method using the factory (`DbFactory`):

```C#
private async Task DeleteContactAsync()
{
    using var context = DbFactory.CreateDbContext();

    if (context.Contacts is not null)
    {
        var contact = await context.Contacts.FirstAsync(...);

        if (contact is not null)
        {
            context.Contacts?.Remove(contact);
            await context.SaveChangesAsync();
        }
    }
}
```

## Scope a database context to the lifetime of the component
Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/blazor-ef-core?view=aspnetcore-9.0#scope-a-database-context-to-the-lifetime-of-the-component
## Enable Sensitive Data Loading
Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/blazor-ef-core?view=aspnetcore-9.0#enable-sensitive-data-logging
