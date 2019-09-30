# Use v. Run v. Map

## Run

```
app.Run(async context => {
    await context.Response.WriteAsync("Hello Liana!");
});
```

## Use

```
app.Use(async (context, next) => {
    await context.Response.WriteAsync("Executing first useeeee\n");
    await next.Invoke();
});

app.Use(async (context, next) => {
    await context.Response.WriteAsync("Executing second useeeee\n");
    await next.Invoke();
});
```

## Map

```
app.Map("/route1", ConfigFromRoute1);
```

```
public static void ConfigFromRoute1(IApplicationBuilder app) {
    app.Run(async context => {
        await context.Response.WriteAsync("Calling config from route 1");
    });
}
```