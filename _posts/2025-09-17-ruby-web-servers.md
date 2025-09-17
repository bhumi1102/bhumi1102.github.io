---
layout: post
title:  "Ruby Web Servers and Concurrency"
---

What does a web server actually *do*? I imagine all of us at least half understand it, let's explore the remaining half! In a [previous post](https://theleafnode.com/), I covered how rack-based web servers like Puma pass an incoming HTTP request to a Rails application by creating an `env` hash. The `env` part is specific to rack-based servers. But the main responsibility of *all* web servers is to:

- Parse the incoming HTTP request - actually read the raw bytes from TCP sockets and parse headers, body, etc. And manage connections.
- Route the request to web applications to execute application logic, then receive and process the response.
- Manage concurrency and resources - schedule work on cores/processes/threads so it can efficiently and reliably handle many clients at once.

We are going to focus on the last point, concurrency, in this post.

>  This post **is** beginner friendly. If you are newish to programming, you should be able to follow along just fine.

One motivating question: when we start the Rails server, we see the output below from Puma. What is the significance of "single mode" and "Min/Max threads"?

```bash
$ rails server
=> Booting Puma
=> Rails 7.1.3 application starting in development 
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Puma version: 6.4.2 (ruby 3.0.1-p64) ("The Eagle of Durango")
*  Min threads: 5
*  Max threads: 5
```

We can see in the Puma source code that there is a [variable `@todo`](https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb#L40) that contains the work to be performed in a Ruby Queue.

Before we get too far, let's review some common terms around concurrency. I think it's useful to be clear about words and their meaning.

## Process, Thread, I/O bound, CPU bound, Concurrency, Latency
A **process** is an independent program in execution with its own memory. Operating systems schedule processes and manage its resources. The word "worker" is used interchangeably with "process" in web servers/concurrency discussions. A **thread** is a smaller unit of work within a process. Threads are light-weight, it's easier to switch between threads and they share memory (all threads within a process).

**Concurrency** is the ability of a system to be able to *handle many tasks at once*, but not literally at the same time. For example, a Puma worker process with 5 threads handles multiple HTTP requests concurrently by interleaving them, running one while another is waiting. So they all make progress but are not running at the same exact time on the CPU. However, if we had multiple CPU cores, then we could process multiple requests simultaneously, one process on each CPU core. That is **parallelism**. More than one CPU-heavy request is executing at the same exact time.

(Some computer science classes make a bigger deal about this distinction. But that's the essence of the difference between concurrency and parallelism.) 

When a CPU is doing work, it's executing instructions that are part of a request, it may come to a point where it needs to wait for some input before it can do more work. For example, if it's waiting for the result of a database query, we say the CPU is waiting on I/O. If the request has a lot of these I/O waits then we say it's **I/O bound**. On the other hand, **CPU bound** requests are those that do a lot of computation on the CPU and are not generally waiting on external input.

One more concept that is useful to know is Ruby's **Global VM Lock (GVL, a.k.a GIL)**. The GVL is a "lock" (a mutax to be more precise) in CRuby (MRI) that ensures only one thread executes Ruby code at a time within a process. This means Ruby threads can run concurrently, but not in parallel — they take turns holding the lock. Why is this needed? Because Ruby's interpreter and VM are not internally thread-safe, meaning bad things would happen if two threads run at the same time, they could corrupt each other's memory or cause race conditions. GVL is not unique to Ruby, Python and other dynamic languages have them too.

>The grocery checkout counter thought experiment (I thought of this while eating lunch): Five registers are open and each is processing one customer's shopping cart at a time. But 5 customers are becoming served *at the same time, in parallel*. Now to extend this analogy to demonstrate concurrency, imagine this (mind you, this would be insane in practice and frustrating for all parties involved. Nonetheless, it is a useful for this thought experiment): There is only one register and it processes 5 customers concurrently, by checking out three items from each customer's cart and rotating through all 5 customers until their cart is empty. Progress is being made in the cart of each customer, work is being done. But there is sharing of the register (and the memory to keep track of which items belong to which customers and which has been processed and which are remaining). Now a given customer can only *leave* once all items in their cart are processed. That's **latency**. How long that takes *depends* on how many customers are sharing the register (threads) and how much stuff they have (nature of work).

You may hear people make a distinction between a web server vs. an application server. But it's a distinction without an interesting difference (as my former coworker used to say about things). Apache and Nginex can be thought of a "web" servers while Puma and Unicorn might be considered "application" servers. Again, not too interesting.

We're done with words now. Back to web servers.

Essentially, the job of a web server is to serve as many clients as possible (with concurrent *or* parallel work) without increasing latency for everyone. While also reducing the amount of time the CPU is idle and waiting. 

So, as with most things in software, there are trade-offs involved. Let's see how Puma and other web servers make these trade-offs.

## Puma vs. Other Web Servers
Puma has been the default server in Rails since Rails 5 in 2016. Before that WebBrick was the default. **WebBrick** was a simple development server though, never recommended for production. Rails developers used Unicorn, Passenger, or Thin in production instead. I recall using Passenger at work in the early 2010's. Unicorn has been popular since then too.

So what is the difference between these web servers?

**Unicorn** was introduced ~2009. It has a multi-process model with a single tread per process. Each request is handled by a dedicated process and it takes advantage of multiple CPU cores. It's simple, robust, great for CPU-bound work. The down side? Not great at handling high numbers of concurrent I/O-bound requests.  Also, inefficient memory usage with many workers.

Unicorn's multi-process model works well with Ruby's Global VM Lock (GVL) because concurrency is achieved via multiple OS processes, not threads. 

**Puma** was designed from the start for concurrent request handling with a thread-pool per worker process. Puma can also be multi-process (like Unicorn) for scaling across CPU cores.

The reason Puma works as the default web server is that web applications in general are more I/O bound (vs. CPU bound). They wait for database requests or API calls.  

Puma also works well with the GVL. While the GVL isn’t released during Ruby CPU work, it _is_ released during I/O waits (like ActiveRecord queries). This means multiple threads within a worker process can still run effectively "in parallel" when one is waiting for I/O.

### Ruby's Concurrency Story - Fibers?
We have seen processes and threads but what are *fibers*? Fibers are a concurrency primitive in Ruby which are more lightweight than threads. Unlike threads:

- They are not scheduled, they only run when you explicitly switch to them. You can do `Fiber.yield` and `Fiber.resume` to pause and resume execution. Execution is cooperative, not preemptive.
- They are very lightweight compared to threads. They are purely user-space constructs inside Ruby and do not map to OS threads.

In 3.0, Ruby introduced a [Fiber Scheduler](https://docs.ruby-lang.org/en/3.2/Fiber/Scheduler.html), which enables Fibers to handle I/O non-blockingly and creates a transparent, event-loop-based concurrency mode. (this part is admittedly handy wavy)

In terms of web servers, **Falcon** is a fiber-first web server that takes advantage of this concurrency model.

We'll pause here with Ruby's concurrency story. Before wrapping up, let's answer the question about what "single mode" and thread count means in the `bin/rails server` output. These values configure number of processes and number of threads as we can see in the Puma config file:

```ruby
# config/puma.rb

max_threads_count = ENV.fetch("RAILS_MAX_THREADS") { 5 }
min_threads_count = ENV.fetch("RAILS_MIN_THREADS") { max_threads_count }
threads min_threads_count, max_threads_count

# Specifies that the worker count should equal the number of processors in production.
if ENV["RAILS_ENV"] == "production"
  require "concurrent-ruby"
  worker_count = Integer(ENV.fetch("WEB_CONCURRENCY") { Concurrent.physical_processor_count })
  workers worker_count if worker_count > 1
end

# ...
```

To start Puma with a thread pool of min 3 and max 10 threads, we can do this:

```bash
RAILS_MIN_THREADS=3 RAILS_MAX_THREADS=10 bin/rails server
```

To configure more than one worker process with Puma, we can set the `WEB_CONCURRENCY` environment variable (in production). Notice that the worker count is set to equal the number of processor cores in production.

I hadn't seen the `"concurrent-ruby"` library before, so I tried it:

```ruby
>> require "concurrent-ruby"
=> true
>> Concurrent.physical_processor_count
=> 6
```

And yup, I'm on a 6 core processor. Cool.

Anyway, wrapping up! There is more to Ruby's concurrency story, and optimizing web servers for performance. I didn't cover Fibers in detail, the Async gem, or Pitchfork, reforking, copy-on-write, and Ractors in Ruby. To be continued (since my goal is for you to be able read each post in a 10 min coffee break, no matter how many hours it takes me to write it, hah!). 