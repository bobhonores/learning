# Middleware Basics

According to Microsoft [^1]:

> Middleware is software that's assembled into an app pipeline to handle requests and responses. Each component:
> - Chooses whether to pass the request to the next component in the pipeline.
> - Can perform work before and after the next component in the pipeline.
>
> Request delegates are used to build the request pipeline. The request delegates handle each HTTP request.
>
> Request delegates are configured using Run, Map, and Use extension methods. An individual request delegate can be specified in-line as an anonymous method (called in-line middleware), or it can be defined in a reusable class. These reusable classes and in-line anonymous methods are middleware, also called middleware components. Each middleware component in the request pipeline is responsible for invoking the next component in the pipeline or short-circuiting the pipeline. When a middleware short-circuits, it's called a terminal middleware because it prevents further middleware from processing the request.

## Run
This extension method enables short-circuiting with a terminal middleware delegate.

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.Run(async context => {
            await context.Response.WriteAsync("Hello Liana!");
        });
    }
}
```

If you have more than one "run" method. The first one will be the only executed. 

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        // This method will be executed
        app.Run (async context => {
            await context.Response.WriteAsync ("First Run");
        });

        // This method won't be executed
        app.Run (async context => {
            await context.Response.WriteAsync ("Second Run");
        });
    }
}
```

## Use
It calls the next desired middleware delegate. 

It uses two parameters: context (HttpContext) and next (RequestDelegate). The next parameter represents the next delegate in the pipeline. You can short-circuit the pipeline by not calling the next parameter.  

``` csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) => {
            await context.Response.WriteAsync("Executing first useeeee\n");
            await next.Invoke();
        });

        app.Use(async (context, next) => {
            await context.Response.WriteAsync("Executing second useeeee\n");
            await next.Invoke();
        });
    }
}
```

## Map
It allows branching of the pipeline path. Inside that branch, you could use Use or Run methods according your needs.

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.Map("/route1", ConfigFromRoute1);
        // Othen middleware configuration
    }

    public static void ConfigFromRoute1(IApplicationBuilder app) {
        app.Run(async context => {
            await context.Response.WriteAsync("Calling config from route 1");
        });
    }
}
```

### MapWhen 
It branches the request pipeline based on the result of the given predicate. Any predicate of type ```Func<HttpContext, bool>``` can be used to map requests to a new branch of the pipeline.

```csharp
public class Startup
{
    private static void HandleBranch(IApplicationBuilder app)
    {
        app.Run(async context =>
        {
            var branchVer = context.Request.Query["branch"];
            await context.Response.WriteAsync($"Branch used = {branchVer}");
        });
    }

    public void Configure(IApplicationBuilder app)
    {
        app.MapWhen(context => context.Request.Query.ContainsKey("branch"),
                               HandleBranch);

        app.Run(async context =>
        {
            await context.Response.WriteAsync("Hello from non-Map delegate. <p>");
        });
    }
}
```

[^1]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-3.0
