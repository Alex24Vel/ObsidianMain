Source: https://learn.microsoft.com/en-us/aspnet/core/signalr/authn-and-authz?view=aspnetcore-9.0

Tags: #copypasted 

2025-04-19 22:34

SignalR can be used with [ASP.NET Core authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity?view=aspnetcore-9.0) to associate a user with each connection. Multiple connections may be associated with a single user.

*If a token expires during the lifetime of a connection, by default the connection continues to work. `LongPolling` and `ServerSentEvent` connections fail on subsequent requests if they don't send new access tokens. For connections to close when the authentication token expires, set [CloseOnAuthenticationExpiration](https://learn.microsoft.com/en-us/aspnet/core/signalr/configuration?view=aspnetcore-9.0#advanced-http-configuration-options).*

### Cookie authentication
In a browser-based app, cookie authentication allows existing user credentials to automatically flow to SignalR connections. When using the browser client, no extra configuration is needed. If the user is logged in to an app, the SignalR connection automatically inherits this authentication.

However, using cookie authentication from the .NET client requires the app to provide an API to exchange authentication data for a cookie.

### Bearer token authentication

The client can provide an access token instead of using a cookie. The server validates the token and uses it to identify the user. This validation is done only when the connection is established. During the life of the connection, the server doesn't automatically revalidate to check for token revocation.

The access token function provided is called before **every** HTTP request made by SignalR. If the token needs to be renewed in order to keep the connection active, do so from within this function and return the updated token. The token may need to be renewed so it doesn't expire during the connection.

In standard web APIs, bearer tokens are sent in an HTTP header. However, SignalR is unable to set these headers in browsers when using some transports. When using WebSockets and Server-Sent Events, the token is transmitted as a query string parameter.

#### Built-in JWT authentication
On the server, bearer token authentication is configured using the [JWT Bearer middleware](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer):

```C#
builder.Services.AddAuthentication(options =>
{
    // Identity made Cookie authentication the default.
    // However, we want JWT Bearer Auth to be the default.
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(options =>
  {
      // Configure the Authority to the expected value for
      // the authentication provider. This ensures the token
      // is appropriately validated.
      options.Authority = "Authority URL"; // TODO: Update URL

      // We have to hook the OnMessageReceived event in order to
      // allow the JWT authentication handler to read the access
      // token from the query string when a WebSocket or 
      // Server-Sent Events request comes in.

      // Sending the access token in the query string is required when using WebSockets or ServerSentEvents
      // due to a limitation in Browser APIs. We restrict it to only calls to the
      // SignalR hub in this code.
      // See https://docs.microsoft.com/aspnet/core/signalr/security#access-token-logging
      // for more information about security considerations when using
      // the query string to transmit the access token.
      options.Events = new JwtBearerEvents
      {
          OnMessageReceived = context =>
          {
              var accessToken = context.Request.Query["access_token"];

              // If the request is for our hub...
              var path = context.HttpContext.Request.Path;
              if (!string.IsNullOrEmpty(accessToken) &&
                  (path.StartsWithSegments("/hubs/chat")))
              {
                  // Read the token out of the query string
                  context.Token = accessToken;
              }
              return Task.CompletedTask;
          }
      };
  });
```

### Cookies vs. bearer tokens

Cookies are specific to browsers. Sending them from other kinds of clients adds complexity compared to sending bearer tokens. Cookie authentication isn't recommended unless the app only needs to authenticate users from the browser client. **Bearer token authentication is the recommended approach when using clients other than the browser client.**