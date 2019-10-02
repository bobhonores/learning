# Threadpool

Creation of thread is something that costs time and resource and thus will be difficult to manage when dealing with a large number of threads. Thread pool is used in this kind of scenario. When you work with thread pool from .NET you queue your work item in thread pool from where it gets processed by an available thread in the thread pool.

But, after work is done this thread doesn’t get shutdown. Instead of shutting down this thread get back to thread pool where it waits for another work item. The creation and deletion of this threads are managed by thread pool depending upon the work item queued in the thread pool. If no work is there in the thread pool it may decide to kill those threads so they no longer consume the resources.

``` csharp
public class Program
{
    static void Main(string[] args)
    {
        ThreadPool.QueueUserWorkItem(Speak);
        System.Console.WriteLine("Hello from Main!");
    }

    private static void Speak(object state)
    {
        System.Console.WriteLine("Hello from Threadpool");
    }
}
```

The output could be
```
Hello from Threadpool
Hello from Main!
```

The delegate definition for Threadpool is 
```csharp
public delegate void WaitCallback(object state);
```

## Passing parameter
QueueUserWorkItem has an overload to insert a parameter to the method.

```csharp
public static void Main()
{
    ThreadPool.QueueUserWorkItem(Speak, "Hello World!");

    Console.WriteLine("Hello from Main!");
}
```