2025-04-19 22:59

Tags: #interfaces

Source: https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement?view=aspnetcore-9.0

# Example

## Use authorization handlers to customize hub method authorization

Source: https://learn.microsoft.com/en-us/aspnet/core/signalr/authn-and-authz?view=aspnetcore-9.0#use-authorization-handlers-to-customize-hub-method-authorization

Consider the example of a chat room allowing multiple organization sign-in via Microsoft Entra ID. 

Anyone with a Microsoft account can sign in to chat, but only members of the owning organization should be able to ban users or view users' chat histories. Furthermore, we might want to restrict some functionality from specific users. 

```C#
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}
```

# References


