# Grand Central Dispatch (GCD) & Dispatch Queues

CPU (Central Processing Unit):

<p align="center">
<img src ="http://i.imgur.com/2nqzMAf.png">
</p>

The CPU in your iPhone contains what is referred to as multiple cores and therefore they can run multiple threads. This allows us to serve more than one task at any given moment. Think of being able to execute different tasks all at the same time to a horse race. All horses are lined up, the gates open and they are all separately doing their own horse thing in an attempt to get to the end of the track.

It's a powerful tool in your tool-belt to be able to break your application into chunks that will be executed in parallel on more than one thread.  

The numbers 0 through 3 represent separate tasks and they are firing off synchronously, one after the other. 0 starts off and 1 will not start off until 0 is complete. When 0 is complete, 1 will fire off. Note that 1 will not fire off until 0 is finished (done means the task is finished). After 1 is finished, 2 will begin and so on.

Synchronous tasks block. Tasks don't execute at the same time, they execute one after the other.

### Synchronous

0  -------------> done  
1  
2  
3  

0  -------------> done  
1  ------> en route  
2  
3  

0  -------------> done  
1  -------------> done  
2  
3  

0  -------------> done  
1  -------------> done  
2  -----> en route  
3  

0  -------------> done  
1  -------------> done  
2  -------------> done  
3  -----> en route  

0  -------------> done  
1  -------------> done  
2  -------------> done  
3  -------------> done



### Asynchronous

0  ----> en route    
1  ------> en route   
2  -----> en route   
3  -------> en route  

0  -------------> done  
1  --------> en route   
2  -------------> done   
3  ------------> en route  

0  -------------> done   
1  -------------> done  
2  -------------> done   
3  ------------> en route

Notice how the section asynchronous has all of our tasks fire off at the same time. It's no guarantee which one will finish first and it's not required that all finish at the same time. 0 and 3 finished first while 1 and 3 were still executing. This is referred to as asynchronous. We have the ability to run a lot more tasks because of this. This is what I had meant by a horse race.

<p align="center">
<img src="http://i.imgur.com/ubOH8sd.png">
</p>

If we had 7 tasks that needed to be completed, it would be much faster to have all 7 occur at the same time. It would take a much greater amount of time to have horse 1 run it's race first and only when it has completed its race, horse 2 would then begin and so on until horse 7 finishes her race.


## Queue

Lets pretend that we're currently in NYC and we just entered one of the most famous delis in town. Upon entering the deli, we notice that no one is waiting in line. We are the first to enter the store.

-X--------------------------  

X represents us being the first on line. Right behind us becomes another person (depicted by O).

-X-O-------------------------  

A third person walks in (labeled as Z)

-X-O-Z------------------------  

Notice how the line is forming. After a few minutes the server comes out and asks that X please step to the counter and place her order. As X was the first person into the store (first on line), then X will be the first to be served. This is referred to as First-in, First-out (FIFO pattern).

Being served: X  
Still in line: -O-Z------------------------  

Z will not be served until O has been served. This is how a queue operates.

We will create three queues labeled as A, B & C. Each of these queues contain some tasks labeled as A-1, A-2 or C-1, C-4, etc.  

A Queue: [A-1, A-2, A-3, A-4, A-5]  
B Queue: [B-1, B-2, B-3, B-4, B-5]  
C Queue: [C-1, C-2, C-3, C-4, C-5]

We have three different queues that need to do some work. In what order will these queues execute their tasks? Should A finish through all of its tasks before B begins? If so, does C then wait for B finishes before it fires off. If we were to adhere to these principals, then we would be executing our queues synchronously. One after the other as if they were waiting in a line.

A: ---> en route  
B:         
C:  


A being a queue, it has multiple tasks that need to be executed. Does it kick off each one one at a time? If so, A would be considered a **serial queue**. This is what a serial queue looks like:

A-1: ----> en route  
A-2:  
A-3:  
A-4:  
A-5:  

A-1: -------------> done   
A-2: ----> en route  
A-3:  
A-4:  
A-5:  

A-1: -------------> done   
A-2: -------------> done   
A-3: ----> en route    
A-4:  
A-5:

If As tasks all ran at the same time, it would be called a **concurrent queue**. It would instead look something like this:

A-1: -----> en route  
A-2: -----------> en route   
A-3: ----> en route    
A-4: ---------> en route  
A-5: -------> en route   



When A has completed all of its tasks (whether or not if it was serial or concurrent queue), B will then begin its tasks. Our queues are being executed synchronously, one after the other.

A: ------------> done    
B: --> en route         
C:

Now that B has fired off, we need to ask the same question. Is B a serial or concurrent queue. Depending on the answer of that question is how the tasks within our serial or concurrent queue will fire off. B in this example is a concurrent queue:

B-1: -----> en route  
B-2: --------->  
B-3: ----> en route    
B-4: ---------> en route  
B-5: -------> en route

Note that C would not begin executing its tasks until B has finished executing all of its tasks.

## Main Thread

TODO: Go into a discussion with images covering the tap of a button.

The main thread is a queue. It's a queue which takes a lot of precedence over other queues that are running to perform the necessary and critical tasks within your application. The name main implies that it's important, and it is. More specifically, the main thread is a dispatch queue.

Dispatch queues are an easy way to perform tasks asynchronously and concurrently in your application. A task is simply some work that your application needs to perform. For example, you could define a task to perform some calculations, create or modify a data structure, process some data read from a file, or any number of things. You define tasks by placing the corresponding code inside either a function or a block object and adding it to a dispatch queue.

A dispatch queue is an object-like structure that manages the tasks you submit to it. All dispatch queues are first-in, first-out data structures. Thus, the tasks you add to a queue are always started in the same order that they were added. GCD (grand central dispatch) provides some dispatch queues for you automatically, but others you can create for specific purposes.

Lets create a serial queue:

```swift
let serialQueue = DispatchQueue(label: "Serial")
```

`serialQueue` is an instance of `DispatchQueue` labeled as "Serial". Think of this like any other queue which works in a FIFO manner. First in, First out. Imagine that you run a deli and you work behind the counter taking peoples orders. Considering that our `serialQueue` is a serial queue and _not_ a concurrent queue it will complete tasks one at a time. This is also how we will run our deli, our deli line works like a serial queue.

Our first customer walks in and they want a cheese sandwich.

```swift
serialQueue.async {
    print("I will take a cheese sandwich.")
}
```

This is how you add a task to a queue. This task is first in line and will execute first. Not only will it execute first, but any tasks that follow will not execute until it has finished.

Someone else has now walked into the deli and wants a coffee.

Here is what our code currently looks like:

```swift
serialQueue.async {
    print("I will take a cheese sandwich.")
}

serialQueue.async {
    print("I will take a coffee.")
}
```

Our line: -A--B--------  

A is first in line and that task is the print statement "I Will take a cheese sandwich". B represents the second task which is the print statement "I will take a coffee".

Lets add more customers to our line that want more things.

```swift
// A
serialQueue.async {
    print("I will take a cheese sandwich.")
}

// B
serialQueue.async {
    print("I will take a coffee.")
}

// C
serialQueue.async {
    print("I will take an egg sandwich with swiss cheese.")
}

// D
serialQueue.async {
    print("I will take a hot chocolate.")
}
```

Lets put all of this work in a table. Because tables are cool and could help explain what's going on.



|  Task | When it will execute  | Sync/Async  | Code
|---|---|---|---|---|
| A  | First  | async  | print("I will take a cheese sandwich.")
| B  | When A is complete  | async  |   print("I will take a coffee.")
| C  | When B is complete  | async  |   print("I will take an egg sandwich with swiss cheese.")
| D  | When C is complete  | async  |   print("I will take a hot chocolate.")

One of the questions you should be asking is "How are they executing in order when you've added these tasks asynchronously to the queue"? Shouldn't they all be firing of at the same time? This is a good time to differentiate between when a specific task is dispatched to a thread and when it is executed. When a task (block of code) is dispatched, that doesn't mean that it's executed as soon as it's dispatched.

Lets create four different threads and number them from 1-4. These threads represent where our code will be executed (our task) and not the order to which these tasks will be executed. The order is dictated by the queue. And considering that our queue is a serial queue, it will retain the order to which these tasks get executed. The fact that these blocks of code are dispatched asynchronously means that they are dispersed amongst different threads immediately. They are not executed immediately. They execute in the order to which they were added to the queue, because they were added to a serial queue and that's how serial queues work.

| Thread | Task | When Task Will executed
|---|---|---|---|
| 1 | B | When A is complete |
| 2 | A | First |
| 3 | D | When C is complete |
| 4 | C | When B is complete |
| Main | User dragging a view| Immediately, not blocked by other tasks on other threads |

As you can see from our table (tables are never wrong), that the queue goes through each item one at a time (in the order in which they were added to the queue) and dispatches each task asynchronously starting with A. Note that this doesn't mean that they will execute asynchronously.

A ---> Dispatched To Thread 2  
B  
C  
D  

A ---> Dispatched to Thread 2  
B ---> Dispatched to Thread 1  
C  
D

A ---> Dispatched to Thread 2  
B ---> Dispatched to Thread 1  
C ---> Dispatched to Thread 4  
D

A ---> Dispatched to Thread 2  
B ---> Dispatched to Thread 1  
C ---> Dispatched to Thread 4  
D ---> Dispatched to Thread 3  

The various tasks have been dispatched in the order to which they were added to the `serialQueue` These tasks get dispatched one after the other and don't block. This is important to note, we will cover this in more depth shortly but for now recognize that these tasks aren't blocking any other thread and this is the case because they are asynchronous tasks (this is because of adding them to the queue as `async`).

The `serialQueue` being a serial queue, executes its tasks in order (no matter if the tasks are `async` or `sync`). The `serialQueue` maintains this order and executes task A and only when A is finished executing does it execute task B.

A ------> en route (on thread 2)   
B  
C  
D  

A -------------------------> done   
B -----> en route (on thread 1)    
C  
D

A -------------------------> done   
B -------------------------> done     
C -----> en route (on thread 4)  
D

A -------------------------> done   
B -------------------------> done     
C -------------------------> done  
D -----> en route (on thread 3)

A -------------------------> done   
B -------------------------> done     
C -------------------------> done  
D -------------------------> done



The main thread predominately handles the UI tasks which allow for the app to react immediately to taps coming from the user. You want to avoid adding tasks to the main thread that will take some time to execute. For example, downloading images is not something you want to run on the main thread. Our `serialQueue` does not interact with the main thread. We interact with the main thread by working with the main queue. We now understand how queues work (somewhat). So your first question with regards to the main queue should be.. is it a serial queue or a concurrent queue? The **main queue** is a serial queue. So the main queue used in iOS applications acts in the same way our serial queue works above.

As we can see, serial queues execute tasks in order. But all we've done so far is add tasks that print some `String` value to the console. Lets mess around with the `sleep()` function to see how our `serialQueue` handles it.

What if we added a sleep function inside of Task C which represents our user who ordered an egg sandwich. We will pretend that this user steps up to the line, looks up our menu but is taking a really long time with her order. You would think after waiting in line that she would have already thought about what it was she wanted to eat. Instead, she was playing candy crush while simultaneously listening to the new ____ album. I'm not sure what would make this joke land, insert current album at your discretion.

```swift
let serialQueue = DispatchQueue(label: "Serial")

// A
serialQueue.async {
    print("I will take a cheese sandwich.")
}

// B
serialQueue.async {
    print("I will take a coffee.")
}

// C
serialQueue.async {
    sleep(10)
    print("I will take an egg sandwich with swiss cheese.")
}

// D
serialQueue.async {
    print("I will take a hot chocolate.")
}
```

With what you know now, how do you think this will play out. Don't forget, even though tasks are labeled from A->D as asynchronous, this only means that they are dispatched asynchronously across different threads. This doesn't dictate as to when the tasks get executed. If we were to run our app, this is what we would see print to the console:

```
I will take a cheese sandwich.
I will take a coffee.
```

Then.. 10 seconds would go by where we would see the following:

```
I will take an egg sandwich with swiss cheese.
I will take a hot chocolate.
```

That's really cool! Task D (being a good customer), waited her turn while C was taking forever ordering her sandwich. This is the behavior of a serial queue. Here is a table summing this up:  


|  Task | When it will execute  | Sync/Async  | Code |
|---|---|---|---|
| A  | First  | async  | print("I will take a cheese sandwich.") |
| B  | When A is complete  | async  |   print("I will take a coffee.") |
| C  | When B is complete  | async  |  sleep(10); print("I will take an egg sandwich with swiss cheese.") |
| D  | When C is complete  | async  |   print("I will take a hot chocolate.") |


You should note that this sleep function happening in Task C does _not_ block the Main Thread. It does not block all threads because it was marked as an asynchronous task. If instead we marked it as synchronous, it would block threads (including the main thread) and freeze our entire app. We will step through this shortly.

In my `ViewController`, I have the following code:

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    createBlueView()
    let gesture = UIPanGestureRecognizer(target: self, action: #selector(drag))
    blueView.addGestureRecognizer(gesture)
}

func createBlueView() {
    blueView = UIView()
    blueView.backgroundColor = UIColor.blue
    view.addSubview(blueView)
    blueView.translatesAutoresizingMaskIntoConstraints = false
    blueView.widthAnchor.constraint(equalToConstant: view.bounds.width * 0.6).isActive = true
    blueView.heightAnchor.constraint(equalToConstant: view.bounds.width * 0.6).isActive = true
    blueView.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    blueView.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
}

func drag(_ sender: UIPanGestureRecognizer) {
    blueView.center = sender.location(in: view)
}
```








Think of everyone then rushing the counter at once but we still would take their orders in order.

-   
-  

-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-  -  
-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-  -  
-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-   
-  
-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-  
-  
-  
-  
-  
-   
-  
-  
-  
-  
-  
-  
-  
-  

















NOTES: Any time-consuming or CPU demanding tasks should run on concurrent or background queues. Maybe this is difficult for new developers to digest and apply, however it’s the way things should be.
