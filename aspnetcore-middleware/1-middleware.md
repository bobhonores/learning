# Middleware Classes [^1]

The prerequisite to write a middleware class are:
- A public constructor with a parameter of type RequestDelegate.
- A public method named Invoke or InvokeAsync. This method must:
    - Return a Task.
    - Accept a first parameter of type HttpContext.

Additional parameters for the constructor and Invoke/InvokeAsync are populated by dependency injection (DI).

```csharp
public class LoggerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;
    
    public LoggerMiddleware(RequestDelegate next, ILoggerFactory factory) {
        _next = next;
        _logger = factory.CreateLogger<LoggerMiddleware>();
    }

    public async Task InvokeAsync(HttpContext context) {
        using (var reader = new StreamReader(context.Request.Body))
        {
            var requestBody = reader.ReadToEnd();
            _logger.LogInformation(requestBody);
        }

        await _next.Invoke(context);
    }
}
```

## Middleware extension method
This allows you to expose the middleware through an extension method, so you can add it in your configuration.

```csharp
public static class LoggerMiddlewareExtension
{
    public static IApplicationBuilder UseLogger(this IApplicationBuilder applicationBuilder) {
        return applicationBuilder.UseMiddleware<LoggerMiddleware>();
    }
}
```

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app) 
    {
        app.UseLogger();
        // More middleware configuration
    }
}
```

## Other scenarios

Middleware is constructed once per application lifetime, but the Invoke method can accept additional parameters. Because of this lifetime rule, we will inject “IClientConfiguration” as a parameter to InvokeAsync method.

```csharp
public class ClientConfigurationMiddleware
{
    private readonly RequestDelegate _next;

    public ClientConfigurationMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context, IClientConfiguration configuration) {
        if (context.Request.Headers.TryGetValue("CLIENTNAME", out StringValues clientName)) {
            configuration.ClientName = clientName.SingleOrDefault();
        } else {
            configuration.ClientName = $"{Guid.NewGuid()}";
            context.Response.Headers.Add("response", "auto");
        }

        configuration.InvokedDateTime = DateTime.UtcNow;

        await _next.Invoke(context);
    }
}
```

```csharp
public static class ClientConfigurationExtension
{
    public static IApplicationBuilder UseClientConfiguration(this IApplicationBuilder applicationBuilder) {
        return applicationBuilder.UseMiddleware<ClientConfigurationMiddleware>();
    }
}
```
[^1]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write?view=aspnetcore-3.0