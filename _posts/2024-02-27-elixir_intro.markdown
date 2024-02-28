---
title:  "First thing to try: Elixir"
date:   2024-02-27 23:54:35 +0000
---

So I’m starting (continuing) to consume my “Things To Try” list and one of the many projects I have half-backed there happened to be written on Elixir https://elixir-lang.org using Phoenix https://www.phoenixframework.org for god knows what reason.

Today I checked my repo and I started this one 3 years ago… That’s an absurd amount of time for getting something in “pending” state and still remembering it!

Anyhow, I knew I was going to pick this up again for a few days so I had been checking the docs and such, the official guides [https://hexdocs.pm/elixir/introduction.html] are great reading material but they lacke hands-on rhythm. On the other hand, Phoenix Up and Running [https://hexdocs.pm/phoenix/up_and_running.html] docs are great but they left too many open questions for me. This happened because I started to consume all these docs before (3 years ago) and never really used much of them, which left me in an intermediate state where I kind of know things but not really which happened to me a lot… 

So I forgot Phoenix for a moment and concentrated my efforts on Elixir first. Elixir is a statically-typed functional language that compiles to BEAM code which is the virtual machine developed to run Erlang programs, which in turn is another functional programming language specifically created to produce highly efficient parallel programs. Erlang was created in the 80s by Ericsson for handling their telecom systems, it’s currently used by CISCO (which spark the claim that “Erlang handled 90% of internet traffic” https://news.ycombinator.com/item?id=17218190), was used initially for WhatsApp, etc…

Although Elixir and Erlang are different languages they are similar https://elixir-lang.org/crash-course.html at many levels, the more important one is that both rely on the BEAM virtual machine so they are highly optimized for parallel executions.

As his older sister (Erlang is an old lady for me), Elixir is being used by some big companies like Discord, but also Slack and Pinterest are in the list. Although It’s true that they have it on some key places as described for the Discord team here [https://medium.com/@siddharth.sabron/how-discord-used-rust-to-scale-elixir-up-to-11-million-concurrent-users-7eb84194aee5] 

Hands on

I wouldn’t try a hello world because it was boring and would not show the point, but also “showing the point” here means implementing a fault tolerant - performance critical distributed system and I don’t want to keep spinning in circles here.

The basic abstraction on an Elixir program is the process, it means that is the norm to model a program as a tree of processes that send messages to one another. So I thought I would start there, after a couple of iterations I have this

{% highlight elixir %}
# Signal tests
#
defmodule SpawnTest do
  def my_proc(should_sleep) do
    if should_sleep do
      IO.puts("Sleeping...")
      Process.sleep(1000)
      IO.puts("Ok ready")
    end
    receive do
      {:info} ->
        IO.puts("All good")
        my_proc(false)
      {:exit} ->
        IO.puts("Adios")
    end
  end
end

IO.puts("Spawning")
pid = spawn(SpawnTest, :my_proc, [true])
IO.puts("Sending messages")
send(pid, {:info})
send(pid, {:exit})
IO.puts("Done")

# Start monitoring `pid`
ref = Process.monitor(pid)

# Wait until the process monitored by `ref` is down.
receive do
  {:DOWN, ^ref, _, _, _} ->
    IO.puts "Process #{inspect(pid)} is down"
end
{% endhighlight %}

Don't expect to understand it, it messy and doesn't make much sense. Some important stuff is:
- Everything is between "do"..."end" blocks
- Those "receive do ... end" blocks are a special syntax for receiving messages
- "spawn()" start a new process and return the process id. It receives the module, the function name with the ":" prefix, and a list with the arguments
- "send()" send messages to a proces id, messages are some kind of collection of names prepended with ":"

That's the important stuff, the rest is boilerplate. Now when ran it produces this output

{% highlight zsh %}
➜  first_app elixir app.exs                                                             
Spawning
Sending messages
Sleeping...
Done
Ok ready
All good
Adios
Process #PID<0.105.0> is down
{% endhighlight %}

which is kind of revealing!
1. It seems that indeed the spawn process is running in parallel, "Done" was printed while :my_proc was "Sleeping..."
2. After "Ok ready" the process appers to receive the messages we sent while it was sleeping. It means it keeps a queue with the messages it could not handled because it was sleeping
3. As soon as the process wakes up it receive all the pending messages
4. The "main" process waits for all they spawned process to finish before terminate

Multiple Hands on

> What if we have processes running on different machiens? would those messages could be sent as easelly?

This is me wondering after trying that small thing before, spoiler alert: it is as simple.
So I digg some more on internet and found how to do this, it's super simple. The only thing I have to do to try it out was start more than one Elixir shell passing in a "hostname" and have some code in one side to call from the other.

I start two terminals, one with session name (--sname) `choso@localhost` and another one with `playa@localhost`.
{% highlight zsh %}
➜  first_app iex --sname choso@localhost                                                                                 
Erlang/OTP 26 [erts-14.2.1] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:1] [jit:ns] [dtrace]

Interactive Elixir (1.16.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(choso@localhost)1> 
{% endhighlight %}

After playing a while I try this on `choso@localhost`

{% highlight elixir %}
iex(choso@localhost)1> pid = Node.spawn_link(:playa@localhost, fn ->
...(choso@localhost)1>   receive do
...(choso@localhost)1>     {:say_hi, choso_pid} -> send(choso_pid, :hello_there)
...(choso@localhost)1>   end
...(choso@localhost)1> end)
#PID<13042.119.0>
iex(choso@localhost)2> pid
#PID<13042.119.0>
iex(choso@localhost)3> Process.info(pid)
** (ArgumentError) errors were found at the given arguments:

  * 1st argument: not a local pid

    :erlang.process_info(#PID<13042.119.0>)
    (elixir 1.16.0) lib/process.ex:839: Process.info/1
    iex:4: (file)
iex(choso@localhost)4> send(pid, {:say_hi, self()})
{:say_hi, #PID<0.115.0>}
iex(choso@localhost)5> flush()
:hello_there
:ok
{% endhighlight %}

I'm freaking out at this point... this is what we are seeing

1. That "Node.spawn_link()" call pushed the code in the "fn -> ..." clusure to the remote node. So it deployed the code by itself and returned a proces id
2. The process id is not a "normal process id", if I try to get more info it just blow up saying "this process is remote"
3. After flushing the pending messages (not sure why that was needed) I got the expected response ":hello_there"
4. The final ":ok" is the standard response message, whatever came before is payload

So I went ahead and killed `playa@localhost`, I don't really like the beach, and try to send another message

{% highlight elixir %}
iex(choso@localhost)6> send pid, {:say_hi, self()}
{:say_hi, #PID<0.115.0>}
iex(choso@localhost)7> flush
:ok
{% endhighlight %}

The remote process was not running but the "link" still report the `:ok` response but without any payload. 

So, now I felt I have a good grasp of what Elixir was about. Now I had to get my hands on Phoenix to start rolling this thing for real!