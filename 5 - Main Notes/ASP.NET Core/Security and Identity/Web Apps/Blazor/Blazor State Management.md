Source: https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management?view=aspnetcore-9.0&pivots=server

Tags: #copypasted 

# Server



## Maintain user state
Server-side Blazor is a stateful app framework. Most of the time, the app maintains a connection to the server. The user's state is held in the server's memory in a _circuit_.

If a user experiences a temporary network connection loss, Blazor attempts to reconnect the user to their original circuit with their original state. However, reconnecting a user to their original circuit in the server's memory isn't always possible.

When a user can't be reconnected to their original circuit, the user receives a new circuit with an empty state. This is equivalent to closing and reopening a desktop app.

## Persist state across circuits
Generally, maintain state across circuits where users are actively creating data, not simply reading data that already exists.

To preserve state across circuits, the app must persist the data to some other storage location than the server's memory. State persistence isn't automatic.

An app can only persist _app state_. UIs can't be persisted, such as component instances and their render trees. Components and render trees aren't generally serializable. To persist UI state, such as the expanded nodes of a tree view control, the app must use custom code to model the behavior of the UI state as serializable app state.

## Where to persist state
### Server-side storage
For permanent data persistence that spans multiple users and devices, the app can use server-side storage. Options include:

- Blob storage
- Key-value storage
- Relational database
- Table storage

After data is saved, the user's state is retained and available in any new circuit.

### URL
For transient data representing navigation state, model the data as a part of the URL. Examples of user state modeled in the URL include:

- The ID of a viewed entity.
- The current page number in a paged grid.

The contents of the browser's address bar are retained:

- If the user manually reloads the page.
- If the web server becomes unavailable, and the user is forced to reload the page in order to connect to a different server.

For information on defining URL patterns with the [`@page`](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor?view=aspnetcore-9.0#page) directive, see [[ASP.NET Core Blazor routing and navigation]]

### Browser storage
For transient data that the user is actively creating, a commonly used storage location is the browser's [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) and [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) collections:

- `localStorage` is scoped to the browser's instance. If the user reloads the page or closes and reopens the browser, the state persists. If the user opens multiple browser tabs, the state is shared across the tabs. Data persists in `localStorage` until explicitly cleared. The `localStorage` data for a document loaded in a "private browsing" or "incognito" session is cleared when the last "private" tab is closed.
- `sessionStorage` is scoped to the browser tab. If the user reloads the tab, the state persists. If the user closes the tab or the browser, the state is lost. If the user opens multiple browser tabs, each tab has its own independent version of the data.

Generally, `sessionStorage` is safer to use. `sessionStorage` avoids the risk that a user opens multiple tabs and encounters the following:

- Bugs in state storage across tabs.
- Confusing behavior when a tab overwrites the state of other tabs.

`localStorage` is the better choice if the app must persist state across closing and reopening the browser.

Caveats for using browser storage:

- Similar to the use of a server-side database, loading and saving data are asynchronous.
- The requested page doesn't exist in the browser during prerendering, so local storage isn't available during prerendering.
- Storage of a few kilobytes of data is reasonable to persist for server-side Blazor apps. Beyond a few kilobytes, you must consider the performance implications because the data is loaded and saved across the network.
- Users may view or tamper with the data. [ASP.NET Core Data Protection](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-9.0) can mitigate the risk. For example, [ASP.NET Core Protected Browser Storage](https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management?view=aspnetcore-9.0&pivots=server#aspnet-core-protected-browser-storage) uses ASP.NET Core Data Protection.

Third-party NuGet packages provide APIs for working with `localStorage` and `sessionStorage`. It's worth considering choosing a package that transparently uses [ASP.NET Core Data Protection](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-9.0). Data Protection encrypts stored data and reduces the potential risk of tampering with stored data. If JSON-serialized data is stored in plain text, users can see the data using browser developer tools and also modify the stored data. Securing trivial data isn't a problem. For example, reading or modifying the stored color of a UI element isn't a significant security risk to the user or the organization. Avoid allowing users to inspect or tamper with _sensitive data_.

#### ASP.NET Core Protected Browser Storage
Protected Browser Storage relies on ASP.NET Core Data Protection and is only supported for server-side Blazor apps.

##### Save and load data within a component
n any component that requires loading or saving data to browser storage, use the [`@inject`](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor?view=aspnetcore-9.0#inject) directive to inject an instance of either of the following:

- `ProtectedLocalStorage`
- `ProtectedSessionStorage`

To persist the `currentCount` value in the `Counter` component of an app based on the [Blazor project template](https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure?view=aspnetcore-9.0), modify the `IncrementCount` method to use `ProtectedSessionStore.SetAsync`:

```C#
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

In larger, more realistic apps, storage of individual fields is an unlikely scenario. Apps are more likely to store entire model objects that include complex state. `ProtectedSessionStore` automatically serializes and deserializes JSON data to store complex state objects.

In the preceding code example, the `currentCount` data is stored as `sessionStorage['count']` in the user's browser. The data isn't stored in plain text but rather is protected using ASP.NET Core Data Protection. The encrypted data can be inspected if `sessionStorage['count']` is evaluated in the browser's developer console.

To recover the `currentCount` data if the user returns to the `Counter` component later, including if the user is on a new circuit, use `ProtectedSessionStore.GetAsync`:

```C#
protected override async Task OnInitializedAsync()
{
    var result = await ProtectedSessionStore.GetAsync<int>("count");
    currentCount = result.Success ? result.Value : 0;
}
```

If the component's parameters include navigation state, call `ProtectedSessionStore.GetAsync` and assign a non-`null` result in `OnParametersSetAsync`**, not** `OnInitializedAsync`. `OnInitializedAsync` is only called once when the component is first instantiated. 

`OnInitializedAsync` isn't called again later if the user navigates to a different URL while remaining on the same page. 

For more information, see [[ASP.NET Core Razor component lifecycle]].

The examples in this section only work if the server doesn't have prerendering enabled. With prerendering enabled, an error is generated explaining that JavaScript interop calls cannot be issued because the component is being prerendered.

Either disable prerendering or add additional code to work with prerendering. To learn more about writing code that works with prerendering, see the [[#Handle prerendering]] section.

##### Handle the loading state
Since browser storage is accessed asynchronously over a network connection, there's always a period of time before the data is loaded and available to a component. For the best results, render a message while loading is in progress instead of displaying blank or default data.

One approach is to track whether the data is `null`, which means that the data is still loading. In the default `Counter` component, the count is held in an `int`. [Make `currentCount` nullable](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types) by adding a question mark (`?`) to the type (`int`):

```C#
private int? currentCount;
```

Instead of unconditionally displaying the count and **`Increment`** button, display these elements only if the data is loaded by checking [HasValue](https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.hasvalue):

```C#
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

##### Handle prerendering
During prerendering:

- An interactive connection to the user's browser doesn't exist.
- The browser doesn't yet have a page in which it can run JavaScript code.

`localStorage` or `sessionStorage` aren't available during prerendering. If the component attempts to interact with storage, an error is generated explaining that JavaScript interop calls cannot be issued because the component is being prerendered.

One way to resolve the error is to disable prerendering. This is usually the best choice if the app makes heavy use of browser-based storage. Prerendering adds complexity and doesn't benefit the app because the app can't prerender any useful content until `localStorage` or `sessionStorage` are available.

To disable prerendering, indicate the render mode with the `prerender` parameter set to `false` at the highest-level component in the app's component hierarchy that isn't a root component.
```C#
<Routes @rendermode="new InteractiveServerRenderMode(prerender: false)" />
<HeadOutlet @rendermode="new InteractiveServerRenderMode(prerender: false)" />
```
Prerendering might be useful for other pages that don't use `localStorage` or `sessionStorage`. To retain prerendering, defer the loading operation until the browser is connected to the circuit. The following is an example for storing a counter value...

##### Factor out state preservation to a common provider
https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management?view=aspnetcore-9.0&pivots=server#factor-out-state-preservation-to-a-common-provider

### In-memory state container service

### [[Cascading values and parameters]]