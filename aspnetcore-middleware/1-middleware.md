# Middleware

- A public constructor with a parameter of type RequestDelegate.
- A public method named Invoke or InvokeAsync. This method must return a Task and accept the first parameter of type HttpContext. Additional parameters for the Invoke/InvokeAsync are populated by dependency injection (DI).

Middleware is constructed once per application lifetime. 

## Logger Middleware

```
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

```
public static class LoggerMiddlewareExtension
{
    public static IApplicationBuilder UseLogger(this IApplicationBuilder applicationBuilder) {
        return applicationBuilder.UseMiddleware<LoggerMiddleware>();
    }
}
```

## Custom Header Middleware

The Invoke method can accept additional parameters. Because of this lifetime rule, we will inject “IClientConfiguration” as a parameter to InvokeAsync method.

```
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

```
public static class ClientConfigurationExtension
{
    public static IApplicationBuilder UseClientConfiguration(this IApplicationBuilder applicationBuilder) {
        return applicationBuilder.UseMiddleware<ClientConfigurationMiddleware>();
    }
}
```