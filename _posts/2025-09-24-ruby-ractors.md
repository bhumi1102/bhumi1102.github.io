---
layout: post
title:  "Ractors in Ruby, Parallelism, and Performance"
---

{% if subscriber.metadata.first_name %}
Hey {{ subscriber.metadata.first_name }},
{% else %}
Hi there,
{% endif %}

What are Ractors, how do they enable parallelism, and how does that help with performance? Let's explore!

>Coincidently, Rails Worlds 2025 talks were published last week and I noticed that the closing keynote by Aaron Patterson is about this stuff.
>
>Think of this post as a primer for the talk. There is a lot to unpack in that presentation and if you read the below before watching the talk, you may get more out of the talk. I'm going to focus solely on the "What are Ractors?" (and the related ""*why* are Ractors?") question.

Ractors were introduced in Ruby 3.0, but they have been an experimental feature for a while. However, it appears that the Ruby core team is working diligently to make the implementation more robust for the upcoming Ruby 3.5 release.

## Motivation for Ractors
In a [previous post](https://theleafnode.com/ruby-web-servers/), we learned about Ruby's Gobal VM Lock (GVL) and how only one CPU-bound task can execute at a time. The GVL is held during CPU work and it's released during I/O work. So we can achieve concurrency (by that we mean interleaving tasks) but not true parallelism, where multiple tasks are being executed at literally the same time on multiple CPU cores. 

The main construct for doing work in parallel we have is an operating system level Process (in Ruby and in other languages). Each Process will have its own GVL and can execute in parallel on different CPU cores, without interfering with or having to wait for any other Process. Processes have an overhead though, we can't always create new Processes fast enough to take advantage of the parallelism.

Could we get parallelism without paying the cost of an OS-level Process?

Also, what can we do about the GVL? The GVL is needed because Ruby programs are not tread-safe (by default) and have mutable state. Can we have something that has its own GVL but is not as heavy to spin up as a OS-level process? 

This is the goal of Ractors.

**Ractors** (stands for Ruby Actors) have these characteristics:

- Each Ractor has its own GVL and memory space (unlike threads, which all fight for the same GVL). 
- Ractors don’t share Ruby objects directly. Communication between Ractors happens via message passing.
- You pass immutable data (or copies) between Ractors.

Given these characteristics, multiple Ractors can safely run Ruby code in _true parallel_ on different CPU cores.

Now, let's see some code.

## How to Use Ractors
How do we create new Ractors? What does the code look like for this message passing thing?

We can **create a new Ractor** with `Ractor.new` and pass in a block:

```ruby
r = Ractor.new do
    # This block will be run in parallel with other ractors.
  end

# You can name a Ractor with `name:` argument.
r = Ractor.new name: 'my-ractor' do
end
```

We can communicate between Ractors with `send`, `receive`, `yield` and `take` methods. 

**`Ractor.send`** sends a message into the Ractor’s inbox. Since objects aren't shared, Ruby either copies or freezes the object before delivery. 

**`Ractor.receive`** called inside a Ractor to pull the next message from its inbox. 

**`Ractor.yield`** sends a message from inside a Ractor back to whoever is listening. The object is made safe before transfer (immutable or copied). 

**`Ractor.take`** called outside a Ractor to retrieve the next message that is yielded.

We can see all four methods in the example below:

```ruby
?> worker = Ractor.new do
?>   loop do
?>     n = Ractor.receive   # wait for a message
?>     Ractor.yield n * n   # send result back
?>   end
>> end
=> #<Ractor:#3 (try-ractors):4 running>
>> worker.send(2)
=> #<Ractor:#3 (try-ractors):4 running>
>> worker.send(3)
=> #<Ractor:#3 (try-ractors):4 running>
>> worker.take
=> 4
>> worker.take
=> 9
```

All of these methods exist because Ractors don’t share mutable state. The only way they interact is by safely passing messages. The [Ractor docs](https://docs.ruby-lang.org/en/master/ractor_md.html) have the latest details. 

## Comparing Ractor Performance
Let's see some code using Ractors and compare it to some sequential code first.

```ruby
def slow_square(n)
  sleep(1)   # simulate expensive work
  n * n
end

r1 = Ractor.new { slow_square(2) }
r2 = Ractor.new { slow_square(3) }
r3 = Ractor.new { slow_square(4) }

puts [r1.take, r2.take, r3.take].inspect
```

Since there is a 1 second sleep in the `slow_square` method, calling it three times should take about 3 seconds. But with Ractors it takes 1.25 seconds, which means some things *are running in parallel*:

```bash
$ time ruby ractors.rb
ractors.rb:6: warning: Ractor is experimental, and the behavior may change in future versions of Ruby! Also there are many implementation issues.
[4, 9, 16]

real  0m1.250s
user  0m0.102s
sys 0m0.065s
```

Can we compare Ractors to Threads? Will this example demonstrate a difference?

```ruby
def run_sequential
  (2..4).map { |n| slow_square(n) }
end

def run_with_threads
  threads = []
  results = []

  (2..4).each_with_index do |n, i|
    threads << Thread.new { results[i] = slow_square(n) }
  end

  threads.each(&:join)
  results
end

def run_with_ractors
  ractors = []

  (2..4).each do |n|
    ractors << Ractor.new(n) do |m|
      slow_square(m)
    end
  end

  ractors.map(&:take)
end
```

Not a big difference between threads and Ractors:

```bash
[4, 9, 16]
Sequential took: 3.01s

[4, 9, 16]
Threads took: 1.0s

[4, 9, 16]
Ractors took: 1.01s
```

It is surprising that Threads and Ractors in the above example took the same time, ~1 second. Why is that? 

The GVL blocks on Ruby code execution, but it is released during many I/O operations, including `sleep`! So when we call `sleep(1)` in our `slow_square` method, the GVL is actually released, allowing other threads to use the CPU to do their work. So the three method calls are handled concurrently, one executing while one is sleeping. They are not executing at the same time (in parallel) but concurrency still helps the overall time in this tiny example. 

I tried to modify the experiment by having the "slow" method do actual computation (CPU work) instead of just sleeping. I could not produce a substantial difference between Threads and Ractors in these small examples. This likely points to the overhead of creating Ractors, which is less than creating Processes but still greater than creating Threads.

Also by the way, with the latest version of Ruby you will get the following helpful (and accurate, as confirmed by Arron Patterson's presentation) warning:

> ractors.rb:6: warning: Ractor is experimental, and the behavior may change in future versions of Ruby! Also there are many implementation issues.

## Try This Yourself
I did the experiment in the Rails World presentation to compare Threads and Ractors to sequential CPU-heavy work (i.e. calculating Fibonacci). However, the results on my laptop (Intel 6 core) were mixed.

Here's the code from the presentation (slightly modified) so you can try it yourself and see what results you get:

```ruby
require "benchmark"

def fib(n)
  return n if n < 2
  fib(n - 1) + fib(n - 2)
end

def run_sequential_baseline
  5.times.map do |i|
    fib(35)
  end
end

def test_threads
  threads = 5.times.map do |i|
    Thread.new do
      fib(35)
    end
  end

  threads.map(&:value)
end

def test_ractors
  ractors = 5.times.map do |i|
    Ractor.new do
      fib(35)
    end
  end

  ractors.map(&:take)
end

puts Benchmark.realtime { p run_sequential_baseline }
puts Benchmark.realtime { p test_threads }
puts Benchmark.realtime { p test_ractors }
```

For me, Ractors were much slower than Threads and Threads were comparable to sequential work. Which is surprising. Most likely explanation is that the overhead of creating Ractors is not amortized in these small examples.

To wrap up, the promise of Ractors is true parallelism with CPU-heavy work, in a web application it could be something like JSON parsing. Each Ractor has its own GVL + isolated memory and can execute Ruby code simultaneously on different CPU cores. Ractors are still experimental but keep an eye out for Ruby 3.5 :)
