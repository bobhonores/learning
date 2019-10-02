# Async



## Why Async? [^1]

##For Web Developer
If you build web apps using .NET technologies - the answer is simple: Scalability.

When you make I/O calls - database queries, file reading, reading from HTTP requests, etc. - the thread that is handling the current HTTP request is just waiting.

That’s it. It’s just waiting for a result to come back from the operating system.

Performing a database query, for example, ultimately asks the operating system to connect to the database, send a message and get a message in return. But that is the OS making these requests - not your app.




[^1]: https://www.blog.jamesmichaelhickey.com/Async-Await-For-The-Rest-Of-Us/