Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/location-of-javascript?view=aspnetcore-9.0

Date: 06.05.2025 09:30

Use JS modules because the approach with global functions pollutes the client.

The following `JsCollocation2` component's `OnAfterRenderAsync` method loads a JS module into `module`, which is an [IJSObjectReference](https://learn.microsoft.com/en-us/dotnet/api/microsoft.jsinterop.ijsobjectreference) of the component class. `module` is used to call the `showPrompt2` function. The `{PATH}` placeholder is the path to the component.

```C#
@page "/js-collocation-2"
@implements IAsyncDisposable
@inject IJSRuntime JS

<PageTitle>JS Collocation 2</PageTitle>

<h1>JS Collocation Example 2</h1>

<button @onclick="ShowPrompt">Call showPrompt2</button>

@if (!string.IsNullOrEmpty(result))
{
    <p>
        Hello @result!
    </p>
}

@code {
    private IJSObjectReference? module;
    private string? result;

    protected async override Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            /*
                Change the {PATH} placeholder in the next line to the path of
                the collocated JS file in the app. Examples:

                ./Components/Pages/JsCollocation2.razor.js (.NET 8 or later)
                ./Pages/JsCollocation2.razor.js (.NET 7 or earlier)
            */
            module = await JS.InvokeAsync<IJSObjectReference>("import",
                "./{PATH}/JsCollocation2.razor.js");
        }
    }

    public async Task ShowPrompt()
    {
        if (module is not null)
        {
            result = await module.InvokeAsync<string>(
                "showPrompt2", "What's your name?");
            StateHasChanged();
        }
    }

    async ValueTask IAsyncDisposable.DisposeAsync()
    {
        if (module is not null)
        {
            try
            {
                await module.DisposeAsync();
            }
            catch (JSDisconnectedException)
            {
            }
        }
    }
}
```

In the preceding example, [JSDisconnectedException](https://learn.microsoft.com/en-us/dotnet/api/microsoft.jsinterop.jsdisconnectedexception) is trapped during module disposal in case Blazor's SignalR circuit is lost. If the preceding code is used in a Blazor WebAssembly app, there's no SignalR connection to lose, so you can remove the `try`-`catch` block and leave the line that disposes the module (`await module.DisposeAsync();`). For more information, see [ASP.NET Core Blazor JavaScript interoperability (JS interop)](https://learn.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/?view=aspnetcore-9.0#javascript-interop-calls-without-a-circuit).
