2025-04-19 20:14

Tags: #dotnet #aspnet #blazor #auth #security

Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0&tabs=visual-studio

---
Blazor uses the existing ASP.NET Core authentication mechanisms to establish the user's identity.

If authorization rule enforcement must be guaranteed, don't implement authorization checks in client-side code. Build a Blazor Web App that only relies on server-side rendering (SSR) for authorization checks and rule enforcement.

[[Razor Pages authorization conventions]] **don't apply to routable Razor components**. If a [[non-routable Razor component is embedded in a page of a Razor Pages app]], the page's authorization conventions indirectly affect the Razor component along with the rest of the page's content.

## Securely maintain sensitive data and credentials

Outside of local development testing, we recommend avoiding the use of environment variables to store sensitive data, as environment variables aren't the most secure approach. Environment variables are generally stored in plain, unencrypted text. If the machine or process is compromised, environment variables can be accessed by untrusted parties.

### Secure authentication flows
Source: https://learn.microsoft.com/en-us/aspnet/core/security/?view=aspnetcore-9.0#secure-authentication-flows:

 For local development testing, the [Secret Manager tool](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-9.0) is recommended for securing sensitive data. For more information, see the following resources:

Configuration data guidelines:

- Never store passwords or other sensitive data in configuration provider code or in plain text configuration files. The [Secret Manager](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-9.0) tool can be used to store secrets in development.
- Don't use production secrets in development or test environments.
- Specify secrets outside of the project so that they can't be accidentally committed to a source code repository.

## Antiforgery support
Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0&tabs=visual-studio#antiforgery-support

## Server-side Blazor authentication
Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0&tabs=visual-studio#server-side-blazor-authentication

Server-side Blazor apps are configured for security in the same manner as ASP.NET Core apps.

The authentication context is only established when the app starts, which is when the app first connects to the WebSocket over a SignalR connection ([[Authentication and authorization in ASP.NET Core SignalR]]) with the client. 

Authentication can be based on a cookie or some other bearer token, but authentication is 
managed via the SignalR hub and entirely within the [[Circuit]]. The authentication context is maintained for the lifetime of the circuit. Apps periodically revalidate the user's authentication state every 30 minutes.

If the app must capture users for custom services or react to updates to the user, see **[[ASP.NET Core server-side and Blazor Web App additional security scenarios]]**. ==IMPORTANT==

==IMPORTANT==
Blazor differs from a traditional server-rendered web apps that make new HTTP requests with cookies on every page navigation. Authentication is checked during navigation events. However, cookies aren't involved. Cookies are only sent when making an HTTP request to a server, which isn't what happens when the user navigates in a Blazor app. During navigation, the user's authentication state is checked within the Blazor circuit, which you can update at any time on the server using the [`RevalidatingAuthenticationStateProvider` abstraction](https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0&tabs=visual-studio#additional-security-abstractions).
==IMPORTANT==

Implementing a custom `NavigationManager` to achieve authentication validation during navigation **isn't recommended**. If the app must execute custom authentication state logic during navigation, use a [[Custom Authentication State Provider]]

The built-in or custom [AuthenticationStateProvider](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.authorization.authenticationstateprovider) service obtains authentication state data from ASP.NET Core's [HttpContext.User](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.httpcontext.user). This is how authentication state integrates with existing ASP.NET Core authentication mechanisms.

### Shared state
Server-side Blazor apps live in server memory, and multiple app sessions are hosted within the same process. For each app session, Blazor starts a circuit with its own dependency injection container scope, thus scoped services are unique per Blazor session.

Warning

We don't recommend apps on the same server share state using singleton services unless extreme care is taken, as this can introduce security vulnerabilities, such as leaking user state across circuits.

You can use stateful singleton services in Blazor apps if they're specifically designed for it. For example, use of a singleton memory cache is acceptable because a memory cache requires a key to access a given entry. Assuming users don't have control over the cache keys that are used with the cache, state stored in the cache doesn't leak across circuits.

For more info refer to [[Blazor State Management]]
### Server-side security of sensitive data and credentials

In test/staging and production environments, server-side Blazor code and web APIs should use secure authentication flows that avoid maintaining credentials within project code or configuration files. Outside of local development testing, we recommend avoiding the use of environment variables to store sensitive data, as environment variables aren't the most secure approach. For local development testing, the [Secret Manager tool](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-9.0)  - [[Safe storage of app secrets in development in ASP.NET Core]] is recommended for securing sensitive data. For more information, see the following resources:

- [Secure authentication flows (ASP.NET Core documentation)](https://learn.microsoft.com/en-us/aspnet/core/security/?view=aspnetcore-9.0#secure-authentication-flows)
- [Managed identities for Microsoft Azure services (Blazor documentation)](https://learn.microsoft.com/en-us/aspnet/core/blazor/security/?view=aspnetcore-9.0#managed-identities-for-microsoft-azure-services)

### [[Blazor Identity UI (Individual Accounts)]]
