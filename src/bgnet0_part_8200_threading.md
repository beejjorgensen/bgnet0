# Appendix: Multithreading {#appendix-threading}

_Multithreading_ is the idea that a process can have multiple _threads_
of execution. That is, it can be running a number of functions at the
same time, as it were.

This is really useful if you have a function that is waiting for
something to happen, and you need another function to keep running at
the same time.

This would be useful in a multiuser chat client because it has to do two
things at once, both of which block:

* Wait for the user to type in their chat message.
* Wait for the server to send more messages.

If we didn't use multithreading, we wouldn't be able to receive messages
from the server while we were waiting for user input and vice versa.

> Side note: `select()` actually has the capability to add regular file
> descriptors to the set to listen for. So it technically _could_ listen
> for data on the sockets **and** the keyboard. This doesn't work in
> Windows, however. And the design of the client is simpler overall with
> multithreading.

Let's take a look at how this works in Python.

## Concepts

There are a few terms we should get straight first.

* **Thread**: a representation of a "thread of execution", that is, a
  part of the program that is executing at this particular moment. If
  you want multiple parts of the program to execute at the same time,
  you can place them in separate threads.

* **Main Thread**: this is the thread that is running by default. We
  just never named it before. But the code that you run without thinking
  about threads is technically running in the main thread.

* **Spawning**: We say we "spawn" a new thread to run a particular
  function. If we do this, the function will execute _at the same time_
  as the main thread. They'll both run at once!

* **Join**: A thread can wait for another to exit by calling that
  thread's `join()` method. This conceptually joins the other thread
  back up to the calling thread. Usually this is the main thread
  `join()`ing the threads it spawned back to itself.

* **Target**: The thread target is a function that the thread will run.
  When this function returns, the thread exits.

It's important to note that _global objects are shared between threads_!
This means one thread can set the value in a global object and other
threads will see those changes. You don't have to worry about this if
the shared data is read-only, but do have to consider this if it's
writable.

We are about to get into the weeds with concurrency and synchronization
here, so for this project, let's just not use any global shared objects.
Remember the old proverb:

> Shared Nothing Is Happy Everybody

That's not really an old proverb. I just made it up. And it sounds kind
of selfish now that I read it again.

Back on track to a related notion: local objects are _not_ shared
between threads. This means threads get their own local variables and
parameter values. They can change them and those changes will not be
visible to other threads.

Also, if you have multiple threads running at the same time, the order
in which they are executed is unpredictable. This really only gets to be
a problem if there is some kind of timing or data dependency between
threads, and again we're starting to get out in the weeds. Let's just be
aware that the order of execution is unpredictable and that'll be OK for
this project.

That should be enough to get started.

## Multithreading In Python

Let's write a program that spawns three threads.

Each thread will run a function called `runner()` (you can call the
function whatever you wish). This function takes two arguments: a `name`
and a `count`. It loops and prints the `name` out `count` times.

The thread exits when the `runner()` function returns.

You can create a new thread by calling the `threading.Thread()`
constructor.

You can run a thread with its `.start()` method.

And you can wait for the thread to complete with its `.join()` method.

Let's take a peek!

<!-- read in the projects/threaddemo.py file here -->
``` {.py}
import threading
import time

def runner(name, count):
    """ Thread running function. """

    for i in range(count):
        print(f"Running: {name} {i}")
        time.sleep(0.2)  # seconds

# Launch this many threads
THREAD_COUNT = 3

# We need to keep track of them so that we can join() them later. We'll
# put all the thread references into this array
threads = []

# Launch all threads!!
for i in range(THREAD_COUNT):

    # Give them a name
    name = f"Thread{i}"

    # Set up the thread object. We're going to run the function called
    # "runner" and pass it two arguments: the thread's name and count:
    t = threading.Thread(target=runner, args=(name, i+3))

    # The thread won't start executing until we call `start()`:
    t.start()

    # Keep track of this thread so we can join() it later.
    threads.append(t)

# Join all the threads back up to this, the main thread. The main thread
# will block on the join() call until the thread is complete. If the
# thread is already complete, the join() returns immediately.

for t in threads:
    t.join()
```

And here's the output:

``` {.default}
Running: Thread0 0
Running: Thread1 0
Running: Thread2 0
Running: Thread1 1
Running: Thread0 1
Running: Thread2 1
Running: Thread1 2
Running: Thread0 2
Running: Thread2 2
Running: Thread1 3
Running: Thread2 3
Running: Thread2 4
```

They're all running at the same time!

Notice the execution order isn't consistent. It might ever vary from run
to run. And that's OK for this program since the threads don't depend on
one another.

## Daemon Threads

Python classifies threads in two different ways:

* Regular, normal, run-of-the-mill threads
* Daemon threads (pronounced "DEE-mun" or "DAY-mun")

The general idea is that a daemon thread will keep running forever and
never return from its function. Unlike non-daemon threads, these threads
will be automatically killed by Python once all the non-daemon threads
are dead.

### This is Somehow Related to `CTRL-C`

If you kill the main thread with `CTRL-C` and there are no other
non-daemon threads running, all daemon threads will also be killed.

But if you have some non-daemon threads, you have to `CTRL-C` through
all of them before you get back to the prompt.

In the final project, we'll be running a thread forever to listen for
incoming messages from the server. So that should be a daemon thread.

You can create a daemon thread like this:

``` {.py}
t = threading.Thread(target=runner, daemon=True)
```

Then at least `CTRL-C` will get you out of the client easily.

## Reflect

* Describe the type of problem using threads would solve.

* What's the difference between a daemon and non-daemon thread in
  Python?

* What do you have to do to create the main thread in Python, if
  anything?

## Threading Project

If you're looking to flex your muscles a bit, here's a little project to
work on.

### What We're Building

The client who has hired us in this case has several ranges of numbers.
They want the sum total of all the sums of all the ranges.

For example, if the ranges are:

``` {.py}
[
    [1,5],
    [20,22]
]
```

We want to:

* First add up `1+2+3+4+5` to get `15`.
* Then add up `20+21+22` to get `63`.
* Then add `15+63` to get `78`, the final answer.

They want the range sums and the total sum printed out. For the example
above, they'd want it to print:

``` {.default}
[15, 63]
78
```

### Overall Structure

The program MUST use threads to solve the problem because the client
really likes parallelism.

You should write a function that adds up a range of numbers. Then you'll
spawn a thread for each range and have that thread work on that range.
If there are 10 ranges of numbers, there will be 10 threads, one
processing each range.

The main thread will:

* Allocate an array for the results. This array length should the same
  as the number of ranges (which is the same as the number of threads).
  Each thread has its own slot to store a result in the array.

  ``` {.py}
  result = [0] * n   # Create an array of `n` zeros
  ```

* In a loop, launch all the threads. Thread arguments are:

  * The thread ID number `0..(n-1)`, where `n` is the number of threads.
    This is the index the thread will use to store its result in the
    result array.

  * The starting value of the range.

  * The ending value of the range.

  * The result array, where the thread will store the result.

* The main thread should keep track of all the thread objects returned
    from `threading.Thread()` in an array. It'll need them in the next
    step.

* In another loop after that, call `.join()` on all the threads. This
  will cause the main thread to wait until all the subthreads have
  completed.

* Print out the results. After all the `join()`s, the result array will
  have all the sums in it.

### Useful Functions

* `threading.Thread()`: create a thread.

* `range(a, b)`: produces all the integers in the range `[a, b)` as an
  iterable.

* `sum()`: compute the sum of an iterable.

* `enumerate()`: produces indexs and values over an iterable.

### Things the Thread Running Function Needs

All the threads are going to write into a shared array. This array can
be set up ahead of time to have zeros in all the elements. There should
be one element per thread, so that each thread can fill in the proper
one.

To make this work, you'll have to pass the thread's index number to its
run function so it knows which element in the shared array to put the
result in!

### Example Run

Example input (you can just hardcode this in your program):

``` {.py}
ranges = [
    [10, 20],
    [1, 5],
    [70, 80],
    [27, 92],
    [0, 16]
]
```

Corresponding output:

``` {.default}
[165, 15, 825, 3927, 136]
5068
```

### Extensions

* If you're using `sum()` or a `for`-loop, what's your time complexity?

* The closed-form equation for the sum of integers from 1 to _n_ is
  `n*(n+1)//2`. Can you use that to get a better time complexity? How
  much better?


<!-- Rubric:

10
Threads spawned, one thread per range

5
Main thread join()s will all spawned threads

5
Each thread properly populates a results array

5
Correct array of sums is outputted 

5
Correct complete total is outputted

-->
