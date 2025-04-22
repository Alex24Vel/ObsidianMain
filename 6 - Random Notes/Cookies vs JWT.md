Date: 2025-04-19 22:54

Sources: 
- https://learn.microsoft.com/en-us/aspnet/core/signalr/authn-and-authz?view=aspnetcore-9.0#cookie-authentication
- 

Cookies are specific to browsers. Sending them from other kinds of clients adds complexity compared to sending bearer tokens. Cookie authentication isn't recommended unless the app only needs to authenticate users from the browser client. **Bearer token authentication is the recommended approach when using clients other than the browser client.**
