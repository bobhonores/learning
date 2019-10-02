# Threads

It represents an actual OS-level thread, which its own stack and kernel resources. But what is a thread? 

> **According to Wikipedia**, a thread or thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler.

> **According to Jon Skeet**, it is a sort of context in which code is running.

> **According to GeeksforGeeks**, it is a lightweight process, or in other words, a thread is a unit which executes the code under the program.

Let's write some code. BTW, Thread class is inside the namespace ```System.Threading```.

```csharp
static void Main (string[] args) {
    ThreadStart job = new ThreadStart (ThreadJob);
    Thread thread = new Thread (job);
    thread.Start ();

    for (int i = 0; i < 5; i++) {
        Console.WriteLine ("Main thread: {0}", i);
        Thread.Sleep (1000);
    }
}

static void ThreadJob () {
    for (int i = 0; i < 10; i++) {
        Console.WriteLine ("Other thread: {0}", i);
        Thread.Sleep (500);
    }
}
```
The output of the above will be:
```
Main thread: 0
Other thread: 0
Other thread: 1
Other thread: 2
Main thread: 1
Other thread: 3
Other thread: 4
Main thread: 2
Other thread: 5
Other thread: 6
Main thread: 3
Other thread: 7
Other thread: 8
Main thread: 4
Other thread: 9
```

## Passing Parameters to Threads
Since ThreadStart delegate doesn't take any parameters, so in order to pass the information it must be stored in somewhere else if you're going to create a thread by yourself. 

One way to do this is by creating an instance of a class, and storing the information inside of it. Often the class itself can contain the delegate used for starting the thread.

Let's look the sample code below

```csharp
public class UrlFetcher {
    private string url;

    public UrlFetcher (string url) {
        this.url = url;
    }

    public void Fetch () {
        Console.WriteLine ($"This url is being downloaded: {this.url}");
        Thread.Sleep (10000);
        System.Console.WriteLine ($"The url {this.url} was downloaded.");
    }
}

class Program {
    static void Main (string[] args) {
        var fetcher = new UrlFetcher ("https://www.google.com");
        new Thread (new ThreadStart (fetcher.Fetch)).Start ();
    }
}
```

The output
```
This url is being downloaded: https://www.google.com
The url https://www.google.com was downloaded.
```

### Using ParameterizedThreadStart
This delegate takes a parameter of type object and it could be used instead of ThreadStart. When we call the method Thread.Start we could pass the parameter as argument.

Cons of this approach are the only accepts a single parameter and it isn't type-safe.

``` csharp
[In some method or other]
Thread t = new Thread (new ParameterizedThreadStart(FetchUrl));
t.Start (myUrl);

[And the actual method...]
static void FetchUrl(object url)
{
    // use url here, probably casting it to a known type before use
}
```

## Joining Threads

It allows to wait until another thread completes its execution.

```csharp
public class ExThread  
{  
  
    // Non-Static method 
    public void mythread()  
    {  
        for (int x = 0; x < 4; x++)  
        {  
            Console.WriteLine(x);  
            Thread.Sleep(100);  
        }  
    }  
  
    // Non-Static method 
    public void mythread1() 
    { 
        Console.WriteLine("2nd thread is Working.."); 
    } 
}  
  
// Driver Class 
public class ThreadExample  
{  
    // Main method 
    public static void Main()  
    {  
        // Creating instance for 
        // mythread() method 
        ExThread obj = new ExThread();  
          
        // Creating and initializing threads  
        Thread thr1 = new Thread(new ThreadStart(obj.mythread));  
        Thread thr2 = new Thread(new ThreadStart(obj.mythread1));  
        thr1.Start();  
          
        // Join thread 
        thr1.Join();  
        thr2.Start();  
          
    }  
}  
```
The output
```
0
1
2
3
2nd thread is Working..
```