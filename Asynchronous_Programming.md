###### Synchronous Programming

- Concept: tasks need to be done in sequence: `A` - `B` - `C` - `D`

- `B` cannot be executed until `A` finishes



###### Benefits of Asynchronous Programming

1. User interface more responsive: in synchronous programming, user interface:
   
   - Will be locked up when a task is waiting to be completed

2. Parallel tasks: in asynchronous programming, if multiple tasks are not relevant:
   
   - They can be executed in parallel

    

###### Demo (1 / 2)

We'll use a demo project to show the effect of *asynchronous programming*

- 2 buttons on the form, representing *synchronous* & *asynchronous* respectively

- These 2 buttons execute the same work, in their own way (*synchronous* or not)

- The work to do is just access websites, by `WebClient.DownloadString` method
  
  - Only `WebClient` is a built-in `.NET` class, all others are self-defined

- When a button is clicked, the work is executed, and the time to do that:
  
  - Will be shown in the textbox underneath

    

###### Demo (2 / 2)

Here's the overall screenshot of the demo

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-06-15-22-18-image.png" title="" alt="" width="317">    

    

###### Synchronous Programming

Here's our code for the "*synchronous*" button:

```csharp
private void btnSync_Click(object sender, EventArgs e)
{
    var watch = System.Diagnostics.Stopwatch.StartNew();

    RunDownloadSync();

    watch.Stop();
    var elaspedMs = watch.ElapsedMilliseconds;
    resultWindow.Text += $"Total execution time: {elaspedMs}ms";
}
```

```csharp
private void RunDownloadSync()
{
    List<String> websites = PrepData();

    foreach(String website in websites)
    {
        WebsiteDataModel model = DownloadWebsite(website);
        ReportWebsiteInfo(model);
    }
}
```

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-06-16-25-01-image.png" title="" alt="" width="485">

- Notice after we click the "*synchronous*" button, if we drag the form, it doesn't move

- Also all the text in the box are shown up all at once, rather than one by one

    

###### Asynchronous Programming (1 / 4)

What we modified for the "*asynchronous*" button, is to just add an `A` to it

```csharp
private void btnSync_Click(object sender, EventArgs e)
{
    ...
    RunDownloadAsync();
    ...
}
```

For `RunDownloadAsync`, we'll call `Task.Run` method, and add `await` to the front

```csharp
private void RunDownloadAsync()
{
    ...
    WebsiteDataModel model = await Task.Run(() => 
        DownloadWebsite(website));
    ...
}
```

    

###### Await (1 / 2)

- The `await` operator suspends the evaluation of the `[?]` enclosing method, until:
  
  - The asynchronous operation represented by its operand completes

- Why do we have `await` in *asynchronous programming*, it seems no sense that:
  
  - We need to wait for sth to complete, which is opposite to *asynchronous*

- It's because this *synchronous* behavior only exists in the very 1st enclosing method
  
  - For any other method, they could go on, with this certain method stuck
  
  - We'll see an example of this behavior very soon

    

###### Await (2 / 2)

Why error messages?

<img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-06-16-48-29-image.png" alt="" width="588">

- `await` can only be used in an `async` method, which'd be added to the signature

- Notice `[?]` above? *Asynchronous* should be filled in. All *asynchronous* methods:
  
  - Should have `async` operator to it, and have `async` as postfix to its name

- Also never have `void` as the return type for any `async` methods. Can return: 
  
  - A `Task`, or what you plan to return enclosed in a `Task` (e.g., `Task<string>`)

    

###### Asynchronous Programming (2 / 4)

Let's click the "*asynchronous*" button & see what's gonna happen

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-06-16-53-36-image.png" title="" alt="" width="474">

- The *total execution time* is printed first! No `await` before `RunDownloadAsync`
  
  <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-06-16-55-55-image.png" title="" alt="" width="468">

- That's we we mentioned as "*an example of this behavior*" above, the `await`:
  
  - In `RunDownloadAsync` doesn't affect `btnAsync_Click`

- The green underline has hinted us what to add: another `await` & `async`
  
  ![](C:\Users\yangs\AppData\Roaming\marktext\images\2024-04-06-16-58-45-image.png)

    

###### Comparison

- If you run the demo right now, by clicking 2 buttons, you'll find the total time:
  
  - Is almost the same, and the *synchronous* version is even a little faster!

- The benefits that *asynchronous* is parallel and thus faster, is gone! Need change!
  
  - The reason this is slow is that, all websites aren't accesed in parallel
  
  - Before each `DownloadWebsite`, there is an `await`, meaning no other access:
    
    - Before the current one gets finished!

    

###### Asynchronous Programming (3 / 4)

Now we want to get our tasks done in parallel, but how?

- First, put all tasks into a basket (i.e., `Task<WebsiteDataModel>`), remove `await`

- Then when everything in the basket gets done (), report them

- Let's view the new version of accessing, called `RunDownloadAsyncInParallel`

    

###### RunDownloadAsyncInParallel

```csharp
private async Task RunDownloadAsyncInParallel()
{
    List<String> websites = PrepData();
    var tasks = new List<Task<WebsiteDataModel>>();

    foreach (string website in websites)
    {
        tasks.Add(Task.Run(() => DownloadWebsite(website)));
    }

    var results = await Task.WhenAll(tasks);
    foreach (var result in results)
    {
        ReportWebsiteInfo(result);
    }
}
```

    

###### Reflection

Q: what does `Task.Run` do?

- Transform a synchronous method call into an asynchronous (or *awaitable*) one

- What if we have the `DownloadWebsite` call `async` directly?

    

###### Asynchronous Programming (4 / 4)

Here's an asynchronous version of `DownloadWebsite` & `RunDownloadAsyncInParallel`

```csharp
private async Task<WebsiteDataModel> DownloadWebsiteAsync(string website)
{
    WebsiteDataModel model = new WebsiteDataModel();
    model.WebsiteURL = website;

    WebClient client = new WebClient();
    model.WebsiteData = await client.DownloadStringTaskAsync(website);
    return model;
}
```

```csharp
private async Task RunDownloadAsyncInParallel()
{
    List<String> websites = PrepData();
    var tasks = new List<Task<WebsiteDataModel>>();

    foreach (string website in websites)
    {
        tasks.Add(DownloadWebsiteAsync(website));
    }

    var results = await Task.WhenAll(tasks);
    foreach (var result in results)
    {
        ReportWebsiteInfo(result);
    }
}
```

    

###### Summary

- Whenever you have an `await` in an enclosing method, the method'd be `async` 

- Whenever a method becomes `async`, its return type'd be enclosed in a `Task<>`

- You can await an `async` method call, or not. But if not, it'll 1st return a `Task`
  
  - Enclosing the output type you want, then let your code keep going 

- When we talk about parallel programming, we think about threading contacts
  
  - Pattern models, context switching, etc. Forget it, only 3 things:
  
  - `async`, `await`, `Task`


