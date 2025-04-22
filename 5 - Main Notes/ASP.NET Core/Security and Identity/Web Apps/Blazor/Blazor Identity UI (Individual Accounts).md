Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0&tabs=visual-studio#blazor-identity-ui-individual-accounts

Tags: #copypasted

The Blazor Web App template scaffolds Identity code for a SQL Server database. The command line version uses SQLite and includes a SQLite database for Identity.

The template:

- Supports interactive server-side rendering (interactive SSR) and client-side rendering (CSR) scenarios with authenticated users.
- Adds Identity Razor components and related logic for routine authentication tasks, such as signing users in and out. The Identity components also support advanced Identity features, such as [[Account Confirmation and Password Recovery]] and [[Multi-factor authentication in ASP.NET Core]] using a third-party app. Note that the Identity components themselves don't support interactivity.
- Adds the Identity-related packages and dependencies.
- References the Identity packages in `_Imports.razor`.
- Creates a custom user Identity class (`ApplicationUser`).
- Creates and registers an EF Core database context (`ApplicationDbContext`).
- Configures routing for the built-in Identity endpoints.
- Includes Identity validation and business logic.

When you choose the Interactive WebAssembly or Interactive Auto render modes, the server handles all authentication and authorization requests, and the Identity components render statically on the server in the Blazor Web App's main project.

The framework provides a custom [AuthenticationStateProvider](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.authorization.authenticationstateprovider) in both the server and client (`.Client`) projects to flow the user's authentication state to the browser. The server project calls `AddAuthenticationStateSerialization`, while the client project calls `AddAuthenticationStateDeserialization`. Authenticating on the server rather than the client allows the app to access authentication state during prerendering and before the .NET WebAssembly runtime is initialized. The custom [AuthenticationStateProvider](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.authorization.authenticationstateprovider) implementations use the [Persistent Component State service](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/prerender?view=aspnetcore-9.0#persist-prerendered-state) ([PersistentComponentState](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.persistentcomponentstate)) to serialize the authentication state into HTML comments and then read it back from WebAssembly to create a new [AuthenticationState](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.authorization.authenticationstate) instance. For more information, see the [Manage authentication state in Blazor Web Apps](https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0&tabs=visual-studio#manage-authentication-state-in-blazor-web-apps) section.


