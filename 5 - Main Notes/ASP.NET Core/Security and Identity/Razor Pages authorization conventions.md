# Razor Pages authorization conventions in ASP.NET Core

Source: https://learn.microsoft.com/en-us/aspnet/core/security/authorization/razor-pages-authorization?view=aspnetcore-9.0

Relevance: low 

---

One way to control access in your Razor Pages app is to use authorization conventions at startup. These conventions allow you to authorize users and allow anonymous users to access individual pages or folders of pages. The conventions described in this topic automatically apply [[authorization filters]] to control access.

Sample code that uses [[Cookie Authentication without ASP.NET Core Identity]]. The concepts and examples shown in this topic apply equally to apps that use ASP.NET Core Identity. To use ASP.NET Core Identity, follow the guidance in [[Introduction to Identity on ASP.NET Core]].

## Require authorization to access a page

Use the [AuthorizePage](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage) convention to add an [AuthorizeFilter](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) to the page at the specified path:


``` C#
services.AddRazorPages(options =>
{
    options.Conventions.AuthorizePage("/Contact");
    options.Conventions.AuthorizeFolder("/Private");
    options.Conventions.AllowAnonymousToPage("/Private/PublicPage");
    options.Conventions.AllowAnonymousToFolder("/Private/PublicPages");
});
```

The specified path is the View Engine path, which is the Razor Pages root relative path without an extension and containing only forward slashes.

To specify an [authorization policy](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-9.0), use an [AuthorizePage overload](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage):


``` C#
options.Conventions.AuthorizePage("/Contact", "AtLeast21");
```

Note

An [AuthorizeFilter](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) can be applied to a page model class with the `[Authorize]` filter attribute. For more information, see [[Authorize filter attribute]]

## Require authorization to access a folder of pages

Use the [AuthorizeFolder](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizefolder) convention to add an [AuthorizeFilter](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) to all of the pages in a folder at the specified path:

```C#
services.AddRazorPages(options =>
{
    options.Conventions.AuthorizePage("/Contact");
    options.Conventions.AuthorizeFolder("/Private");
    options.Conventions.AllowAnonymousToPage("/Private/PublicPage");
    options.Conventions.AllowAnonymousToFolder("/Private/PublicPages");
});
```

The specified path is the View Engine path, which is the Razor Pages root relative path.

To specify an [authorization policy](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-9.0), use an [AuthorizeFolder overload](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizefolder):

```C#
options.Conventions.AuthorizeFolder("/Private", "AtLeast21");
```
