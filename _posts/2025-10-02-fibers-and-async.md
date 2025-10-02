---
layout: post
title:  "Ruby Fibers and Async"
---

{% if subscriber.metadata.first_name %}
Hey {{ subscriber.metadata.first_name }},
{% else %}
Hi there,
{% endif %}

In a [previous post](), we saw that the goal with Ractors is to achieve parallelism and optimize for CPU-heavy work. What if we wanted to optimize for IO-heavy work, where the CPU is waiting for the results of some external work? Ruby Fibers have this goal. 

> You can [read this online]().
> 
> Also, this post is intended to be beginner friendly. We will define terms enough to get a practical sense of what's what (but not in a pure computer science or academic way).

## What are Fibers?
Fibers have been around for a while, since Ruby 1.9 but become more practical after Ruby 3.0. Fibers are one way to do concurrency in Ruby. Let's unpack the below from the [docs](https://docs.ruby-lang.org/en/master/Fiber.html):

*Fibers are primitives for implementing light weight cooperative concurrency in Ruby. Basically they are a means of creating code blocks that can be paused and resumed, much like threads. The main difference is that they are never preempted and that the scheduling must be done by the programmer and not the VM.*

What is "light-weight" about Fibers? Multiple Fibers run on one OS Thread. They are cheap to create, cheap to switch to, and have lower memory overhead (compared to Threads). Fibers operate in "user space" and are managed by Ruby rather than the OS (i.e. "kernal space").

What is "**cooperative concurrency**"? It means that tasks voluntarily yield control so that other tasks can run. Nothing interrupts them automatically (as opposed to preemptive scheduling of Threads where the Operating System preempts them). The Ruby VM will not preemptively pause a Fiber the way the OS does with Threads.

Ruby Fibers have methods like `yield` and `resume`. The code running inside the Fiber can call `Fiber.yield` and give control back to its caller. When a Fiber is created with `Fiber.new`, it does not run automatically. Rather it must be explicitly asked to run using the `Fiber#resume` method. 

Fibers return the value of the last executed expression, upon yielding or termination.

So the `yield` method is what's use to release control in a "I'm done for now, here you can go next" way vs. the Operating System scheduler coming in and saying "Your time's up. I'm pausing you and letting someone else go". 

The below code demonstrates `resume` and `yield`:

```ruby
?> fiber = Fiber.new do
?>   Fiber.yield 1
?>   2
>> end
>> 
>> puts fiber.resume
>> puts fiber.resume
>> puts fiber.resume
1
2
(try-fibers):8:in 'Fiber#resume': attempt to resume a terminated fiber (FiberError)
  from (try-fibers):8:in '<main>'
```

## Fiber Scheduler And Async Gem
Ruby 3.0 introduced the concept of a Fiber scheduler with the `Fiber::Scheduler` class.

The goal of a Fiber Scheduler is to make I/O non-blocking. What does it mean for something to be **non-blocking**? Let's use `sleep()` as an example. 

Normally, when we call `sleep(1)` in Ruby, the entire thread is blocked for 1 second. Nothing else in that thread can run until `sleep` finishes (Side note: but other threads *can* run as I/O waits do release the GVL <-- connecting this to [this post](https://theleafnode.com/ruby-web-servers/)). If you had multiple Fibers, they’d all be stuck waiting.

The Fiber Scheduler turns `sleep` (and other I/O waits) into non-blocking operations. It intercepts blocking operations such as `sleep` and says "okay, this Fiber is waiting 1 second. Let’s yield back to the scheduler and run another Fiber in the meantime." 

As a result, our program can have hundreds or thousands of Fibers, all “sleeping” or waiting on I/O, and the scheduler can juggle them cooperatively on a single thread.

Note that the `Fiber::Scheduler` in Ruby is an abstract class. It doesn't magically make `sleep` non-blocking. We need something that implements the scheduler. (i.e. if you try `Fiber.set_scheduler(Fiber::Scheduler.new)` that won't work)

The `Async` gem [implements](https://github.com/socketry/async/blob/eb873e4a5c300f4d80ad6345894dc50277126a5c/lib/async/scheduler.rb#L28) the `Fiber::Scheduler` [interface](https://docs.ruby-lang.org/en/master/Fiber/Scheduler.html). So we can install that gem and make our `sleep` non-blocking.

But wait a minute, what's `Async`? It's "an awesome asynchronous event-driven reactor for Ruby" according to the gem's description. In the concurrency world, the word "reactor" refers to a central event loop that watches for I/O and timers, and resumes tasks which are ready to execute. We can think of the `Async` "reactor" as the "scheduler". (The concept of an "event loop" is similar to how concurrency in Node.js works btw).

Let's see how to use `Async` to demonstrate Fibers and non-blocking I/O with a code example.  

## Fiber Example
Using the `Async` gem, we demonstrate that `sleep()` is non-blocking and yields to other tasks:

```ruby
require "async"

# Simulate an I/O-bound task
def fake_io_task(id)
  puts "[Fiber #{id}] starting"
  sleep(1)  # yields to the scheduler instead of blocking
  puts "[Fiber #{id}] done"
end

puts "Running Fibers concurrently with Async"

Async do |task|
  fibers = 5.times.map do |i|
    task.async { fake_io_task(i + 1) }
  end

  # Wait for all tasks to finish
  fibers.each(&:wait)
end

puts "All fibers finished!"
```

When we wrap code in an `Async {...}` block, we’re starting an event loop (i.e. a reactor or a scheduler). The `task` parameter object is the root task that manages everything inside. 

Calling `task.async { ... }` creates a new child task, which runs in its own Fiber. But under the same scheduler/reactor, so multiple tasks can yield on I/O and run concurrently within a single thread. 

Calling `wait` on each task tells the reactor to wait until those child tasks have finished before moving on.

We can put that code snippet in a Ruby file and run it:

```bash
$ ruby fibers.rb 
Running Fibers concurrently with Async
[Fiber 1] starting
[Fiber 2] starting
[Fiber 3] starting
[Fiber 4] starting
[Fiber 5] starting
[Fiber 1] done
[Fiber 2] done
[Fiber 3] done
[Fiber 4] done
[Fiber 5] done
All fibers finished!
```

We can see from the order of the `puts` statements that when Fiber 1 hits the `sleep(1)` line it yields to the scheduler and the "Fiber n is starting" line is printed for others Fibers *before* the "Fiber 1 is done" line is printed after Fiber 1 is resumed by the scheduler (following the `sleep`).

Inside the the `Async` block, Fibers yield cooperatively and many Fibers can run concurrently in one OS thread.

That's the main idea behind Fibers. For the latest on Fibers in Ruby, here's the [Fiber documentation](https://docs.ruby-lang.org/en/master/Fiber.html)

Zooming out, the promise of Fibers is concurrency in an I/O-heavy workload where there is lot of waiting. Since Fibers are "light-weigh" we can have thousands of Fibers in single OS Thread and switch among them "cheaply". Fibers don't run in parallel or on multiple cores though. The other side of this spectrum is running multiple Threads or even multiple OS processes in parallel on multiple CPU cores. That's the direction multi-process Ruby web servers such as Unicorn and Pitchfork take (to be continued :). 
