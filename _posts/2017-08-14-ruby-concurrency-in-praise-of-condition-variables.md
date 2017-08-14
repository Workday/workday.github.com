---
layout: post
title: "Ruby Concurrency: In Praise of Condition Variables"
description: ""
category: "ruby"
tags: [ruby, concurrency]
---
{% include JB/setup %}

_by Tom Van Eyck, Workday Dublin_

When it comes to articles about concurrency, much digital ink has been spilled about the topic of mutexes. While a programmer's familiarity with mutexes is likely to depend on what kind of programs she usually writes, most developers tend to be at least somewhat familiar with these particular synchronization primitives. This article, however, is going to focus on a much lesser-known synchronization construct: the condition variable.

Condition variables are used for putting threads to sleep and waking them back up once a certain condition is met. Don't worry if this sounds a bit vague; we'll go into a lot more detail later. As condition variables always need to be used in conjunction with mutexes, we'll lead with a quick mutex recap. Next, we'll introduce consumer-producer problems and how to elegantly solve them with the aid of condition variables. Then, we'll have a look at how to use these synchronization primitives for implementing blocking method calls. Finishing up, we'll describe some curious condition variable behavior and how to safeguard against it.

<!--more-->
### A mutex recap

A mutex is a data structure for protecting shared state between multiple threads. When a piece of code is wrapped inside a mutex, the mutex guarantees that only one thread at a time can execute this code. If another thread wants to start executing this code, it'll have to wait until our first thread is done with it. I realize this may all sound a bit abstract, so now is probably a good time to bring in some example code.

#### Writing to shared state

In this first example, we'll have a look at what happens when two threads try to modify the same shared variable. The snippet below shows two methods: `counters_with_mutex` and `counters_without_mutex`. Both methods start by creating a zero-initialized `counters` array before spawning 5 threads. Each thread will perform 100,000 loops, with every iteration incrementing all elements of the `counters` array by one. Both methods are the same in every way except for one thing: only one of them uses a mutex.

```ruby
def counters_with_mutex
  mutex = Mutex.new
  counters = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

  5.times.map do
    Thread.new do
      100000.times do
        mutex.synchronize do
          counters.map! { |counter| counter + 1 }
        end
      end
    end
  end.each(&:join)

  counters.inspect
end

def counters_without_mutex
  counters = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

  5.times.map do
    Thread.new do
      100000.times do
        counters.map! { |counter| counter + 1 }
      end
    end
  end.each(&:join)

  counters.inspect
end

puts counters_with_mutex
# => [500000, 500000, 500000, 500000, 500000, 500000, 500000, 500000, 500000, 500000]

puts counters_without_mutex
# => [500000, 447205, 500000, 500000, 500000, 500000, 203656, 500000, 500000, 500000]
# note that we seem to have lost some increments here due to not using a mutex
```

As you can see, only the method that uses a mutex ends up producing the correct result. The method without a mutex seems to have lost some increments. This is because the lack of a mutex makes it possible for our second thread to interrupt our first thread at any point during its execution. This can lead to some serious problems.

For example, imagine that our first thread has just read the first entry of the `counters` array, incremented it by one, and is now getting ready to write this incremented value back to our array. However, before our first thread can write this incremented value, it gets interrupted by the second thread. This second thread then goes on to read the current value of the first entry, increments it by one, and succeeds in writing the result back to our `counters` array. Now we have a problem!

We have a problem because the first thread got interrupted before it had a chance to write its incremented value to the array. When the first thread resumes, it will end up overwriting the value that the second thread just placed in the array. This will cause us to essentially lose an increment operation, which explains why our program output has entries in it that are less than 500,000.

All these problems can be avoided by using a mutex. Remember that a thread executing code wrapped by a mutex cannot be interleaved with another thread wanting to execute this same code. Therefore, our second thread would never have gotten interleaved with the first thread, thereby avoiding the possibility of results getting overwritten.

#### Reading from shared state

There's a common misconception that a mutex is only required when writing to a shared variable, and not when reading from it. The snippet below shows 50 threads flipping the boolean values in the `flags` array over and over again. Many developers think this snippet is without error as the code responsible for changing these values was wrapped inside a mutex. If that were true, then every line of the output of `puts flags.to_s` should consist of 10 repetitions of either `true` or `false`. As we can see below, this is not the case.

```ruby
mutex = Mutex.new
flags = [false, false, false, false, false, false, false, false, false, false]

threads = 50.times.map do
  Thread.new do
    100000.times do
      # don't do this! Reading from shared state requires a mutex!
      puts flags.to_s

      mutex.synchronize do
        flags.map! { |f| !f }
      end
    end
  end
end
threads.each(&:join)
```
```bash
$ ruby flags.rb > output.log
$ grep 'true, false' output.log | wc -l
    30
```

What's happening here is that our mutex only guarantees that no two threads can modify the `flags` array at the same time. However, it is perfectly possible for one thread to start reading from this array while another thread is busy modifying it, thereby causing the first thread to read an array that contains both `true` and `false` entries. Luckily, all of this can be easily avoided by wrapping `puts flags.to_s` inside our mutex. This will guarantee that only one thread at a time can read from or write to the `flags` array.

Before moving on, I would just like to mention that even very experienced people have gotten tripped up by not using a mutex when accessing shared state. In fact, at one point there even was a [Java design pattern](http://www.javaworld.com/article/2074979/java-concurrency/double-checked-locking--clever--but-broken.html) that assumed it was safe to not always use a mutex to do so. Needless to say, this pattern has since been amended.

### Consumer-producer problems

With that mutex refresher out of the way, we can now start looking at condition variables. Condition variables are best explained by trying to come up with a practical solution to the [consumer-producer problem](https://en.wikipedia.org/wiki/Producer-consumer_problem). In fact, consumer-producer problems are so common that Ruby already has a data structure aimed at solving these: [the Queue class](https://ruby-doc.org/core-2.3.1/Queue.html). This class uses a condition variable to implement the blocking variant of its `shift()` method. In this article, we made a conscious decision not to use the `Queue` class. Instead, we're going to write everything from scratch with the help of condition variables.

Let's have a look at the problem that we're going to solve. Imagine that we have a website where users can generate tasks of varying complexity, e.g. a service that allows users to convert uploaded jpg images to pdf. We can think of these users as producers of a steady stream of tasks of random complexity. These tasks will get stored on a backend server that has several worker processes running on it. Each worker process will grab a task, process it, and then grab the next one. These workers are our task consumers.

With what we know about mutexes, it shouldn't be too hard to write a piece of code that mimics the above scenario. It'll probably end up looking something like this.

```ruby
tasks = []
mutex = Mutex.new
threads = []

class Task
  def initialize
    @duration = rand()
  end

  def execute
    sleep @duration
  end
end

# producer threads
threads += 2.times.map do
  Thread.new do
    while true
      mutex.synchronize do
        tasks << Task.new
        puts "Added task: #{tasks.last.inspect}"
      end
      # limit task production speed
      sleep 0.5
    end
  end
end

# consumer threads
threads += 5.times.map do
  Thread.new do
    while true
      task = nil
      mutex.synchronize do
        if tasks.count > 0
          task = tasks.shift
          puts "Removed task: #{task.inspect}"
        end
      end
      # execute task outside of mutex so we don't unnecessarily
      # block other consumer threads
      task.execute unless task.nil?
    end
  end
end

threads.each(&:join)
```

The above code should be fairly straightforward. There is a `Task` class for creating tasks that take between 0 and 1 seconds to run. We have 2 producer threads, each running an endless `while` loop that safely appends a new task to the `tasks` array every 0.5 seconds with the help of a mutex. Our 5 consumer threads are also running an endless `while` loop, each iteration grabbing the mutex so as to safely check the `tasks` array for available tasks. If a consumer thread finds an available task, it removes the task from the array and starts processing it. Once the task had been processed, the thread moves on to its next iteration, thereby repeating the cycle anew.

While the above implementation seems to work just fine, it is not optimal as it requires all consumer threads to constantly poll the `tasks` array for available work. This polling does not come for free. The Ruby interpreter has to constantly schedule the consumer threads to run, thereby preempting threads that may have actual important work to do. To give an example, the above code will interleave consumer threads that are executing a task with consumer threads that just want to check for newly available tasks. This can become a real problem when there is a large number of consumer threads and only a few tasks.

If you want to see for yourself just how inefficient this approach is, you only need to modify the original code for consumer threads with the code shown below. This modified program prints well over a thousand lines of `This thread has nothing to do` for every single line of `Removed task`. Hopefully, this gives you an indication of the general wastefulness of having consumer threads constantly poll the `tasks` array.

```ruby
# modified consumer threads code
threads += 5.times.map do
  Thread.new do
    while true
      task = nil
      mutex.synchronize do
        if tasks.count > 0
          task = tasks.shift
          puts "Removed task: #{task.inspect}"
        else
          puts 'This thread has nothing to do'
        end
      end
      # execute task outside of mutex so we don't unnecessarily
      # block other consumer threads
      task.execute unless task.nil?
    end
  end
end
```

### Condition variables to the rescue

So how we can create a more efficient solution to the consumer-producer problem? That is where condition variables come into play. Condition variables are used for putting threads to sleep and waking them only once a certain condition is met. Remember that our current solution to the producer-consumer problem is far from ideal because consumer threads need to constantly poll for new tasks to arrive. Things would be much more efficient if our consumer threads could go to sleep and be woken up only when a new task has arrived.

Shown below is a solution to the consumer-producer problem that makes use of condition variables. We'll talk about how this works in a second. For now though, just have a look at the code and perhaps have a go at running it. If you were to run it, you would probably see that `This thread has nothing to do` does not show up anymore. Our new approach has completely gotten rid of consumer threads busy polling the `tasks` array.

The use of a condition variable will now cause our consumer threads to wait for a task to be available in the `tasks` array before proceeding. As a result of this, we can now remove some of the checks we had to have in place in our original consumer code. I've added some comments to the code below to help highlight these removals.

```ruby
tasks = []
mutex = Mutex.new
cond_var = ConditionVariable.new
threads = []

class Task
  def initialize
    @duration = rand()
  end

  def execute
    sleep @duration
  end
end

# producer threads
threads += 2.times.map do
  Thread.new do
    while true
      mutex.synchronize do
        tasks << Task.new
        cond_var.signal
        puts "Added task: #{tasks.last.inspect}"
      end
      # limit task production speed
      sleep 0.5
    end
  end
end

# consumer threads
threads += 5.times.map do
  Thread.new do
    while true
      task = nil
      mutex.synchronize do
        while tasks.empty?
          cond_var.wait(mutex)
        end

        # the `if tasks.count == 0` statement will never be true as the thread
        # will now only reach this line if the tasks array is not empty
        puts 'This thread has nothing to do' if tasks.count == 0

        # similarly, we can now remove the `if tasks.count > 0` check that
        # used to surround this code. We no longer need it as this code will
        # now only get executed if the tasks array is not empty.
        task = tasks.shift
        puts "Removed task: #{task.inspect}"
      end
      # Note that we have now removed `unless task.nil?` from this line as
      # our thread can only arrive here if there is indeed a task available.
      task.execute
    end
  end
end

threads.each(&:join)
```

Aside from us removing some `if` statements, our new code is essentially identical to our previous solution. The only exception to this are the five new lines shown below. Don't worry if some of the accompanying comments don't quite make sense yet. Now is also a good time to point out that the new code for both the producer and consumer threads was added inside the existing mutex synchronization blocks. Condition variables are not thread-safe and therefore always need to be used in conjunction with a mutex!

```ruby
# declaring the condition variable
cond_var = ConditionVariable.new
```

```ruby
# a producer thread now signals the condition variable
# after adding a new task to the tasks array
cond_var.signal
```

```ruby
# a consumer thread now goes to sleep when it sees that
# the tasks array is empty. It can get woken up again
# when a producer thread signals the condition variable.
while tasks.empty?
  cond_var.wait(mutex)
end
```

Let's talk about the new code now. We'll start with the consumer threads snippet. There's actually so much going on in these three lines that we'll limit ourselves to covering what `cond_var.wait(mutex)` does for now. We'll explain the need for the `while tasks.empty?` loop later. The first thing to notice about the `wait` method is the parameter that's being passed to it. Remember how a condition variable is not thread-safe and therefore should only have its methods called inside a mutex synchronization block? It is that mutex that needs to be passed as a parameter to the `wait` method.

Calling `wait` on a condition variable causes two things to happen. First of all, it causes the thread that calls `wait` to go to sleep. That is to say, the thread will tell the interpreter that it no longer wants to be scheduled. However, this thread still has ownership of the mutex as it's going to sleep. We need to ensure that the thread relinquishes this mutex because otherwise all other threads waiting for this mutex will be blocked. By passing this mutex to the `wait` method, the `wait` method internals will ensure that the mutex gets released as the thread goes to sleep.

Let's move on to the producer threads. These threads are now calling `cond_var.signal`. The `signal` method is pretty straightforward in that it wakes up exactly one of the threads that were put to sleep by the `wait` method. This newly awoken thread will indicate to the interpreter that it is ready to start getting scheduled again and then wait for its turn.

So what code does our newly awoken thread start executing once it gets scheduled again? It starts executing from where it left off. Essentially, a newly awoken thread will return from its call to `cond_var.wait(mutex)` and resume from there. Personally, I like to think of calling `wait` as creating a save point inside a thread from which work can resume once the thread gets woken up and rescheduled again. Please note that since the thread wants to resume from where it originally left off, it'll need to reacquire the mutex in order to get scheduled. This mutex reacquisition is very important, so be sure to remember it.

This segues nicely into why we need to use `while tasks.empty?` when calling `wait` in a consumer thread. When our newly awoken thread resumes execution by returning from `cond_var.wait`, the first thing it'll do is complete its previously interrupted iteration through the `while` loop, thereby evaluating `while tasks.empty?` again. This actually causes us to neatly avoid a possible race condition.

Let's say we don't use a `while` loop and use an `if` statement instead. The resulting code would then look like shown below. Unfortunately, there is a very hard to find problem with this code. Note how we now need to re-add the previously removed `if tasks.count > 0` and `unless task.nil?` statements to our code below in order to ensure its safe execution.

```ruby
# consumer threads
threads += 5.times.map do
  Thread.new do
    while true
      task = nil
      mutex.synchronize do
        cond_var.wait(mutex) if tasks.empty?

        # using `if tasks.empty?` forces us to once again add this
        # `if tasks.count > 0` check. We need this check to protect
        # ourselves against a nasty race condition.
        if tasks.count > 0
          task = tasks.shift
          puts "Removed task: #{task.inspect}"
        else
          puts 'This thread has nothing to do'
        end
      end
      # using `if tasks.empty?` forces us to re-add `unless task.nil?`
      # in order to safeguard ourselves against a now newly introduced
      # race condition
      task.execute unless task.nil?
    end
  end
end
```

Imagine a scenario where we have:

- two producer threads
- one consumer thread that's awake
- four consumer threads that are asleep

A consumer thread that's awake will go back to sleep only when there are no more tasks in the `tasks` array. That is to say, a single consumer thread will keep processing tasks until no more tasks are available. Now, let's say one of our producer threads adds a new task to the currently empty `tasks` array before calling `cond_var.signal` at roughly the same time as our active consumer thread is finishing its current task. This `signal` call will awaken one of our sleeping consumer threads, which will then try to get itself scheduled. This is where a race condition is likely to happen!

We're now in a position where two consumer threads are competing for ownership of the mutex in order to get scheduled. Let's say our first consumer thread wins this competition. This thread will now go and grab the task from the `tasks` array before relinquishing the mutex. Our second consumer thread then grabs the mutex and gets to run. However, as the `tasks` array is empty now, there is nothing for this second consumer thread to work on. So this second consumer thread now has to do an entire iteration of its `while true` loop for no real purpose at all.

We now find ourselves in a situation where a complete iteration of the `while true` loop can occur even when the `tasks` array is empty. This is a not unlike the position we were in when our program was just busy polling the `tasks` array. Sure, our current program will be more efficient than busy polling, but we will still need to safeguard our code against the possibility of an iteration occurring when there is no task available. This is why we needed to re-add the `if tasks.count > 0` and `unless task.nil?` statements. Especially the latter of these two is important, as otherwise our program might crash with a `NilException`.

Luckily, we can safely get rid of these easily overlooked safeguards by forcing each newly awakened consumer thread to check for available tasks and having it put itself to sleep again if no tasks are available. This behavior can be accomplished by replacing the `if tasks.empty?` statement with a `while tasks.empty?` loop. If tasks are available, a newly awoken thread will exit the loop and execute the rest of its code. However, if no tasks are found, then the loop is repeated, thereby causing the thread to put itself to sleep again by executing `cond_var.wait`. We'll see in a later section that there is yet another benefit to using this `while` loop.

### Building our own Queue class

At the beginning of a previous section, we touched on how condition variables are used by the `Queue` class to implement blocking behavior. The previous section taught us enough about condition variables for us to go and implement a basic `Queue` class ourselves. We're going to create a thread-safe `SimpleQueue` class that is capable of:

- having data appended to it with the `<<` operator
- having data retrieved from it with a non-blocking `shift` method
- having data retrieved from it with a blocking `shift` method

It's easy enough to write code that meets these first two criteria. It will probably end up looking something like the code shown below. Note that our `SimpleQueue` class is using a mutex as we want this class to be thread-safe, just like the original `Queue` class.

```ruby
class SimpleQueue
  def initialize
    @elems = []
    @mutex = Mutex.new
  end

  def <<(elem)
    @mutex.synchronize do
      @elems << elem
    end
  end

  def shift(blocking = true)
    @mutex.synchronize do
      if blocking
        raise 'yet to be implemented'
      end
      @elems.shift
    end
  end
end

simple_queue = SimpleQueue.new
simple_queue << 'foo'

simple_queue.shift(false)
# => "foo"

simple_queue.shift(false)
# => nil
```

Now let's have a look at what's needed to implement the blocking `shift` behavior. As it turns out, this is actually very easy. We only want the thread to block if the `shift` method is called when the `@elems` array is empty. This is all the information we need to determine where we need to place our condition variable's call to `wait`. Similarly, we want the thread to stop blocking once the `<<` operator appends a new element, thereby causing `@elems` to no longer be empty. This tells us exactly where we need to place our call to `signal`.

In the end, we just need to create a condition variable that makes the thread go to sleep when a blocking `shift` is called on an empty `SimpleQueue`. Likewise, the `<<` operator just needs to signal the condition variable when a new element is added, thereby causing the sleeping thread to be woken up. The takeaway from this is that blocking methods work by causing their calling thread to fall asleep. Also, please note that the call to `@cond_var.wait` takes place inside a `while @elems.empty?` loop. Always use a `while` loop when calling `wait` on a condition variable! Never use an `if` statement!

```ruby
class SimpleQueue
  def initialize
    @elems = []
    @mutex = Mutex.new
    @cond_var = ConditionVariable.new
  end

  def <<(elem)
    @mutex.synchronize do
      @elems << elem
      @cond_var.signal
    end
  end

  def shift(blocking = true)
    @mutex.synchronize do
      if blocking
        while @elems.empty?
          @cond_var.wait(@mutex)
        end
      end
      @elems.shift
    end
  end
end

simple_queue = SimpleQueue.new

# this will print "blocking shift returned with: foo" after 5 seconds
# that is to say, the first thread will go to sleep until the second
# thread adds an element to the queue, thereby causing the first thread
# to be woken up again
threads = []
threads << Thread.new { puts "blocking shift returned with: #{simple_queue.shift}" }
threads << Thread.new { sleep 5; simple_queue << 'foo' }
threads.each(&:join)
```

One thing to point out in the above code is that `@cond_var.signal` can get called even when there are no sleeping threads around. This is a perfectly okay thing to do. In these types of scenarios calling `@cond_var.signal` will just do nothing.

### Spurious wakeups

A "spurious wakeup" refers to a sleeping thread getting woken up without any `signal` call having been made. This is an impossible to avoid edge-case in condition variables. It's important to point out that this is not being caused by a bug in the Ruby interpreter or anything like that. Instead, the designers of the threading libraries used by your OS found that allowing for the occasional spurious wakeup [greatly improves the speed of condition variable operations](https://stackoverflow.com/a/8594644/1420382). As such, any code that uses condition variables needs to take spurious wakeups into account.

So does this mean that we need to rewrite all the code that we've written in this article in an attempt to make it resistant to possible bugs introduced by spurious wakeups? You'll be glad to know that this isn't the case as all code snippets in this article have always wrapped the `cond_var.wait` statement inside a `while` loop!

We covered earlier how using a `while` loop makes our code more efficient when dealing with certain race conditions as it causes a newly awakened thread to check whether there is actually anything to do for it, and if not, the thread goes back to sleep. This same `while` loop helps us deal with spurious wakeups as well.

When a thread gets woken up by a spurious wakeup and there is nothing for it to do, our usage of a `while` loop will cause the thread to detect this and go back to sleep. From the thread's point of view, being awakened by a spurious wakeup isn't any different than being woken up with no available tasks to do. So the same mechanism that helps us deal with race conditions solves our spurious wakeup problem as well. It should be obvious by now that `while` loops play a very important role when working with condition variables.

### Conclusion

Ruby's condition variables are somewhat notorious for their poor documentation. That's a shame, because they are wonderful data structures for efficiently solving a very specific set of problems. Although, as we've seen, using them isn't without pitfalls. I hope that this post will go some way towards making them (and their pitfalls) a bit better understood in the wider Ruby community.

I also feel like I should point out that while everything mentioned above is correct to the best of my knowledge, I'm unable to guarantee that absolutely no mistakes snuck in while writing this. As always, please feel free to contact me if you think I got anything wrong, or even if you just want to say hello.
