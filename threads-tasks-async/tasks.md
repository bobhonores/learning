# Task

Tasks have nothing to do with Threads and this is the cause of many misconceptions, especially if you have in your might something like “well a Task is like a lightweight Thread”. Task is not thread. Task does not guarantee parallel execution. Task does not belong to a Thread or anything like that. They are two separate concepts and should be treated as such.

A task is an object that represents some work that needs be done. A Task may or may not be completed. The moment when it completes can be right now or in the future.

```csharp
public class Program {
    static void Main (string[] args) {
        var task = new Task (() => System.Console.WriteLine ("Hello from Task"));
        task.Start ();

        System.Console.WriteLine ("Hello from Main");
        task.Wait ();
    }
}
```

Other way to create a task is with the method ```Task.Run```.

``` csharp
Thread.CurrentThread.Name = "Main";

Task taskA = Task.Run (() => Console.WriteLine ("Hello from taskA."));

Console.WriteLine ("Hello from thread '{0}'.",
    Thread.CurrentThread.Name);
taskA.Wait ();
```

A couple of notes to remark:
- Task doesn't return a value.
- Task<T> encapsulates a value which will return.

```csharp
static void Main(string[] args)
{
    Task<int> t = Task.Run(() =>
    {
        return 32;
    });
    Console.WriteLine(t.Result); 
    Console.ReadLine();
}
```

## Differences Between Task And Thread
 
Here are some differences between a task and a thread.
- The Thread class is used for creating and manipulating a thread in Windows. A Task represents some asynchronous operation and is part of the Task Parallel Library, a set of APIs for running tasks asynchronously and in parallel.
- The task can return a result. There is no direct mechanism to return the result from a thread.
- Task supports cancellation through the use of cancellation tokens. But Thread doesn't.
- A task can have multiple processes happening at the same time. Threads can only have one task running at a time.
- We can easily implement Asynchronous using ’async’ and ‘await’ keywords.
- A new Thread()is not dealing with Thread pool thread, whereas Task does use thread pool thread.

## Parameter in Tasks

There is an overload, when you declare a task in which you could pass a parameter to the task.

```csharp
var task = new Task<int>((value) =>
            {
                int num = (int)value;
                System.Console.WriteLine(num);

                return num * num;
            }, i);
```

## Continuation of tasks

ContinueWith method allows you to establish a flow continue working with tasks all the way up.

```csharp
public static void Main () {
    Task<int> task = new Task<int> (() => {
        Console.WriteLine ("Task on thread {0} started.",
            Thread.CurrentThread.ManagedThreadId);
        Thread.Sleep (3000);
        Console.WriteLine ("Task on thread {0} finished.",
            Thread.CurrentThread.ManagedThreadId);

        return 2 + 3;
    });

    Task<string> task2 = task.ContinueWith<string> (taskInfo => {
        Console.WriteLine ("The result from the task is {0}.",
            taskInfo.Result);
        Console.WriteLine ("Task2 on thread {0} started.",
            Thread.CurrentThread.ManagedThreadId);
        Thread.Sleep (3000);
        Console.WriteLine ("Task2 on thread {0} finished.",
            Thread.CurrentThread.ManagedThreadId);

        return "Miguel";
    });

    task2.ContinueWith (taskInfo => {
        Console.WriteLine ("The result from the second task is {0}.", taskInfo.Result);
    });

    task.Start ();
    Console.WriteLine ("This is the main thread.");
    Console.ReadKey ();
}
```
The output will be something like this.
```
This is the main thread.
Task on thread 4 started.
Task on thread 4 finished.
The result from the task is 5.
Task2 on thread 5 started.
Task2 on thread 5 finished.
The result from the second task is Miguel.
```

## Continuation for different results or scenarios

Sometime you want to execute certain segments or tasks based upon the previous result.

``` csharp
static void Main(string[] args)
{
    Task<int> t = Task.Run(() =>
    {
        return 32;
    });

    t.ContinueWith((i) =>
    {
        Console.WriteLine("Canceled");
    }, TaskContinuationOptions.OnlyOnCanceled);
    
    t.ContinueWith((i) =>
    {
        Console.WriteLine("Faulted");
    }, TaskContinuationOptions.OnlyOnFaulted);
    
    var completedTask = t.ContinueWith((i) =>
    {
        Console.WriteLine(i.Result);
        Console.WriteLine("Completed");
    }, TaskContinuationOptions.OnlyOnRanToCompletion);
    
    Console.ReadLine();
}
```

https://kudchikarsk.com/tasks-in-csharp/csharp-task/
