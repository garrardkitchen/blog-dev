---
title: ".NET Aspire and Redis"
date: 2024-02-25T09:41:08Z
tags: [.net aspire, azd, blazor, csharp, devex, dependency injection, nuget, redis]
---

# First look into .NET Aspire 

TL;DR: [See here](#addendum) for why AZD is not yet supported.

I decided to give .NET Aspire a try. I'm yet to watch an online tutorial but I did watch its announcement a few months ago.  At the time I did think YAGNI.  For me, it didn't make much sense.  Either that, or it just wasn't being explained well enough.  Enthusiasm alone isn't enough to promote a new tool. Since then, I’ve seen many tweets. So, over the last month or so, I’ve gradually grasped its raison d’être.

Last night, with some time to kill, I asked Copilot to send me some YT links so I could learn more. But Copilot refused to share any YT links. It only gave me some MS Learn links. One of them took me to the **[Samples GH repo](https://github.com/dotnet/aspire-samples)**.

The first time I opened a sample project, the Visual Studio Community 2022 (Preview) version of VS I was using, asked me to install an Aspire component. It turned out to be the Aspire workload. It was a simple click-and-forget operation. I wish everything in life was that easy. 

To avoid this ahead of time you can:

{{< hint info >}}

**.NET Aspire workload**

Install the .NET Aspire workload

```powershell
dotnet workload install aspire
```

or update all workloads

```powershell
dotnet workload update
```

then to confirm version and install status

```powershell
dotnet workload list
```

{{</hint >}}

# The `Wow`

The sample I tried to run in Visual Studio did not work, so I switched to another one. Then I saw the dashboard on my personal developer laptop for the first time. I was impressed (a bit). I explored the metrics, logs, and traces. I was more impressed.

I was even more impressed when I saw an example solution that had both a Windows Forms and a WPF app. I wanted to test how easy it was to set up and find a situation that would require me to code.


{{< hint warning >}}
**My sample repo** 

You can find my sample .NET Aspire applicaton here:
https://github.com/garrardkitchen/dotnet-aspire-and-redis-sample

{{</hint>}}

To create your first .NET Aspire application is straight forward.  From the menu you choose `Create a new project`.  I filtered on the .NET Apsire project type just to see what's available:

{{< figure src="../img/2024-02-25-10-49-32.png" alt="" caption=".NET Aspire project templates" >}}

I selected the .NET Aspire Starter Application and named it `sample1`.  I was prompted to start up Docker for Desktop due to the Redis dependency.  The next action I performed was to debug the app to see if anything barked at me.  It didn't and I was presented with the dashboard:

{{< figure src="../img/2024-02-25-10-54-40.png" alt="" caption=".NET Aspire dashboard" >}}

I clicked on the Endpoint link next to the `frontendweb` project and this Blazor app loaded:

{{< figure src="../img/2024-02-25-10-55-19.png" alt="" caption="My first Aspire hosted sample app" >}}

I clicked on the Counter menu option, then the `click me` button which incremented the on page counter.  I clicked away and when I returned the counter had zeroed.  Great, I thought, this is where I can start hacking away to see how easy it is to persist this to the redis cache.  The remainder of this post covers what I did.


# Persisting a Counter to Redis cache

Ref: https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling?tabs=visual-studio


## Redis Dependency

First off, I needed a new pre-release NuGet package for this to work:

```
dotnet add package Aspire.StackExchange.Redis.DistributedCaching --prerelease
```

Next I added this so the necessary types get added to the service collection, by adding the following to Program.cs in sample1.Web :

```csharp
builder.AddRedisDistributedCache("cache");
```

## RedisClient

I then created a Redis Client that I could use to call from the counter razor page to obtain and update the counter value.  This is that code:

👉 _I've not chosen to use the incr type_

👉 _Redis stores these string as a hash types_

```csharp
using Microsoft.Extensions.Caching.Distributed;

namespace sample1.Web;

public class RedisClient(IDistributedCache cache)  
{
    public async Task<int> GetCounterAsync(string key)
    {
        var value = await cache.GetStringAsync(key);
        return value == null ? 0 : int.Parse(value);
    }

    public async Task SetCounterAsync(string key, int value)
    {
        await cache.SetStringAsync(key, value.ToString());
    }
}
```

## Dependency Injection

I then added this Redis Client to the service collection, again, in the this sample1.Web's Program.cs file:

```csharp
builder.Services.AddScoped<RedisClient>();
```

## Razor page

The last piece to this jigsaw is the counter razor page.  There's a few different pieces to do here so to help I've decomposed these to steps.

Step 1: Inject this new RedisClient into the page:

```
@inject RedisClient RedisClient
```

Step 2: Overwrite the page's `OnInitializedAsync` implementation in order for us to be able to get our counter from our redis cache:

```csharp
protected override async Task OnInitializedAsync()
{
    currentCount = await RedisClient.GetCounterAsync("counter");
}
```

Step 3: We now need to update this counter when the button is pressed:

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await RedisClient.SetCounterAsync("counter", currentCount);
}
```

This is what the resulting page looks like this:

```html
@page "/counter"
@rendermode InteractiveServer

@inject RedisClient RedisClient

<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p role="status">Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    protected override async Task OnInitializedAsync()
    {
        currentCount = await RedisClient.GetCounterAsync("counter");
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await RedisClient.SetCounterAsync("counter", currentCount);
    }
}
```

I felt that what I had done was logical and flowed well.  I do have experience [limited] with Blazor pages so I did go into this feeling confident and that this was not going to be too much of a stretch.

Just like that, I was all set to run the app! I'm delighted to announce it worked flawlessly and as intended, like most of the code I write 👀.

## Confirmation of counter persistence

I couldn't leave it there.  I had to make sure the counter was being written to the cache. Nothing fancy, all I did was I exec'ed into the running container, and then ran a few cli commands.

To get the current value of the counter, use:

```powershell
redis-cli hget counter "data"   
```

I also update the counter then refresh the page to see it reflect this change:

```powershell
redis-cli hset counter "data" 25 
```

I also looked at the distributed tracing to get confirmation of this.  Here's an example of this:

{{< figure src="../img/2024-02-25-11-55-37.png" alt="" caption="Distributed trace example" >}}

And that was that. I was able to take a sample Aspire app and perist a counter value to Redis cache with very little effort. All in all, this was a positive DevEx. 

# Conclusion

.NET Aspire appears to offer that _simplicity glue_ that is very much missing and I must admit, it does looks great from an inner loop perspective. I'm yet to evaluate its CD approach and how it handles the provisioning of the supporting infrastructure. I see from [Deploy a .NET Aspire app using AZD](https://learn.microsoft.com/en-us/dotnet/aspire/deployment/azure/aca-deployment-azd-in-depth) that both bicep and Terraform are supported in that scernario.  This will be my next step.  I am also reassured that the tech that I'm using on various projects, as a direct result of my research, such as bicep, TF, AZD and ACA, is being utilized here as well.  I do hope the `wow` and positive DevEx continues.

# Addendum

_I added this section a few days after posting the original article_

Well, I couldn't wait to use AZD to deploy my sample app.  Sadly, this didn't go too well.  

I got this error [condensed] at the deploy application stage:

{{< hint danger>}}

error CONTAINER1013: Failed to push to the output registry: The request was canceled due to the configured HttpClient.Timeout of 100 seconds elapsing

{{</ hint>}}

I managed to deploy most of the infra plus 2 ACAs (`apiserver` and `cache`) with the `azd up` command but failed to deploy `webfrontend`.  On retry, it fails this time with `apiservice` which it had previously deployed successfully.  I retried with `azd deploy webfrontend` but I saw a repeat of the the above.

I found this GH issue that confirms this behaviour ➡️ https://github.com/Azure/azure-dev/issues/3225 and that it's yet to be addressed.

So, sadly you can't use AZD with .NET Aspire, yet.  It's a real shame this hasn't been made public.

# References

- https://github.com/dotnet/aspire-samples
- https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling?tabs=visual-studio
- https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/
- https://learn.microsoft.com/en-us/dotnet/aspire/deployment/azure/aca-deployment-azd-in-depth