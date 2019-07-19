# Cosmos DB as a Globally Distributed Cache Provider

## Introduction

This is a simple .Net Core 2.1 Web MVC to demonstrate how Cosmos DB can be used as a globally distributed cache provider.

## Setup

- Copy the repo
- Create a Cosmos DB with the SQL(Core) API
- Create a Database and a container
- Change the settings in the app.json file
- Run the application

## References

I added the following references to the MVC project:

- Microsoft.Azure.Cosmos

## Code to notice

- Add the Cosmos Cache provider

```c#
// Startup.cs
services.AddCosmosCache((CosmosCacheOptions cacheOptions) =>
{
    cacheOptions.ContainerName = Configuration["CosmosCacheContainer"];
    cacheOptions.DatabaseName = Configuration["CosmosCacheDatabase"];
    cacheOptions.ClientBuilder = new CosmosClientBuilder(Configuration["CosmosConnectionString"]);
    cacheOptions.CreateIfNotExists = true;
});

services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromSeconds(3600);
    options.Cookie.IsEssential = true;
});
```

> Note: The AddCosmosCache services comes from the following link:<br/>https://github.com/Azure/Microsoft.Extensions.Caching.Cosmos


- Test the cosmos cache provider

```c#
// IndexController.cs
public IActionResult Index()
{
    // Put data into Gache
    HttpContext.Session.SetString(SessionKeyName, "Sample Value");
    return View();
}

// cache the response
[ResponseCache(Duration = 2)]
public IActionResult SayTime()
{
    if (string.IsNullOrEmpty(HttpContext.Session.GetString(SessionKeyName)))
    {
        // Get data from Session
        ViewData["Data"] = HttpContext.Session.GetString(SessionKeyName);
        HttpContext.Session.SetString(SessionKeyName, "The Doctor");
    }
    ViewData["Time"] = DateTime.Now.ToString();
    return View();
}
```
