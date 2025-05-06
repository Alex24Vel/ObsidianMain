Source: https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-9.0

Date: 24.04.2025 12:58

Tags:

## Overview

Caching works best with data that changes infrequently **and** is expensive to generate.
Apps should be written and tested to **never** depend on cached data.

ASP.NET Core supports several different caches.

The simplest cache is based on the [[IMemoryCache]]: stored in the memory of the web server.
Apps running on a server farm (multiple servers) should ensure sessions are sticky when using the in-memory cache. Sticky sessions ensure that requests from a client all go to the same server. See: [[Application Request Routing]].

Non-sticky sessions in a web farm require a [[Distributed Caching]] to avoid cache consistency problems. For some apps, a distributed cache can support higher scale-out than an in-memory cache. Using a distributed cache offloads the cache memory to an external process.

The in-memory cache can store any object. The distributed cache interface is limited to `byte[]`. Both cache items as key-value pairs.

## NuGet Package
[Microsoft.Extensions.Caching.Memory](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/) / `**IMemoryCache**` **is recommended** over `System.Runtime.Caching`/`MemoryCache` because it's better integrated into ASP.NET Core. For example, `IMemoryCache` works natively with ASP.NET Core [[Dependency Injection]]

Use `System.Runtime.Caching`/`MemoryCache` as a compatibility bridge when porting code from ASP.NET 4.x to ASP.NET Core.

## Cache guidelines
- Code should always have a fallback option to fetch data and **not** depend on a cached value being available.
- The cache uses a scarce resource, memory. Limit cache growth:
    - Do **not** insert external input into the cache. As an example, using arbitrary user-provided input as a cache key is not recommended since the input might consume an unpredictable amount of memory.
    - Use expirations to limit cache growth.
    - [[## Use SetSize, Size, and SizeLimit to limit cache size]]. The ASP.NET Core runtime does **not** limit cache size based on memory pressure. It's up to the developer to limit cache size.

## Use IMemoryCache
**See: [Use IMemoryCache Section](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-9.0#use-imemorycache)** for whole article, some notes are below:

The following code uses the `Set` extension method to cache data for a relative time without `MemoryCacheEntryOptions`:

`_memoryCache.Set(CacheKeys.Entry, DateTime.Now, TimeSpan.FromDays(1));`

In the preceding code, the cache entry is configured with a relative expiration of one day. The cache entry gets evicted from the cache after one day, even if it's accessed within this timeout period.

Example from a project:
```C#
var cacheKey = $"PasswordRecoveryCode_{user.Email}";
var cacheEntryOptions = new MemoryCacheEntryOptions()
	.SetAbsoluteExpiration(TimeSpan.FromMinutes(15)); //Consistent expiration
_memoryCache.Set(cacheKey, code, cacheEntryOptions);
```
- The simplified `_memoryCache.Set(CacheKeys.Entry, DateTime.Now, TimeSpan.FromDays(1));`  uses **relative expiration**.
- The code from an example project uses **absolute expiration**.

For a password recovery service, using SetAbsoluteExpiration is actually the better choice because:
1. It provides a strict time limit for security purposes
2. The code is more explicit about its intentions
3. It's easier to add more security-related cache options later if needed
4. Using MemoryCacheEntryOptions allows to add more configuration options if needed in the future. For example:
	- Add post-eviction callbacks
	- Set cache priorities
	- Add cache dependencies
	- Combine with sliding expiration
	- Set size limits

A cached item set with only a sliding expiration is at risk of never expiring. If the cached item is repeatedly accessed within the sliding expiration interval, the item never expires. 
Combine a sliding expiration with an absolute expiration to guarantee the item expires. If either the sliding expiration interval _or_ the absolute expiration time pass, the item is evicted from the cache.

## MemoryCacheEntryOptions
The following example:
- Sets the cache priority to [CacheItemPriority.NeverRemove](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.cacheitempriority#microsoft-extensions-caching-memory-cacheitempriority-neverremove).
- Sets a [PostEvictionDelegate](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate) that gets called after the entry is evicted from the cache. The callback is run on a different thread from the code that removes the item from the cache.
```C#
public void OnGetCacheRegisterPostEvictionCallback()
{
    var memoryCacheEntryOptions = new MemoryCacheEntryOptions()
        .SetPriority(CacheItemPriority.NeverRemove)
        .RegisterPostEvictionCallback(PostEvictionCallback, _memoryCache);

    _memoryCache.Set(CacheKeys.CallbackEntry, DateTime.Now, memoryCacheEntryOptions);
}

private static void PostEvictionCallback(
    object cacheKey, object cacheValue, EvictionReason evictionReason, object state)
{
    var memoryCache = (IMemoryCache)state;

    memoryCache.Set(
        CacheKeys.CallbackMessage,
        $"Entry {cacheKey} was evicted: {evictionReason}.");
}
```
## Use SetSize, Size, and SizeLimit to limit cache size
Source: https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-9.0#use-setsize-size-and-sizelimit-to-limit-cache-size

## MemoryCache.Compact
Source: https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-9.0#memorycachecompact

## Cache Dependencies
Source: https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-9.0#cache-dependencies

## Background cache update
Use a [[Background Service]] such as [IHostedService](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihostedservice) to update the cache. The background service can recompute the entries and then assign them to the cache only when they’re ready.
## Additional notes
Source: https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-9.0#additional-notes

