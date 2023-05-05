https://www.youtube.com/watch?v=SquGNt4FhY0

- a web server is an application that accepts connections on a socket, then serves http apps over those connections

- sockets are endpoints for straming data to and from clients, identified by `[IP_addr, port]`. we create a socket for each new connection to our server. each socket has a file descriptor. socktes in puma are either TCP, TCP w/SSL, or Unix

- puma uses mainly the TCPSocket and UnixSocker classes from stdlib

```ruby
require 'socket'

s=TCPSocker.new 'localhost', 2000

while line=s.gets # read lines from socket
	puts line
end

s.close # close socker when done
```

- a rack server is a type of web server that serves Rack-compatible applications writtend in ruby
- rack app are objects which respond to 'call'(with an env argument) and return below
```
[
	200,
	{"Header"=>"Value"},
	["The response body"]
]
``` 
- all rails controller actions are just mini rack application

- the Rack server calls the application with **the environment hash**
```
{
	"REQUEST_METHOD"=>"GET",
	"PATH_INFO"=>"/",
	"HTTP_VERSION"=>"1.1",
	...
}
@app.call(env)
```
- Rack servers are interfaces between Rack-compatible Ruby applications and a socket
- Socket -> HTTP -> The Rack Env
- suck the HTTP off of Socket, turn it into a ruby object(아마 Env) and then hand that ruby object in an very specific format off to this application

- at this point, we have a mental model of how puma works
![](/assets/mental-model.png)

- we've got a socket, socket has some http on it, we've turned the http into a rack environment and the rack env gets passed to the rack application

- puma, like almost all ruby web servers, is a 'pre-forking' web server.
- all we do is we start one process up(main process), the main process starts up and it boots the application optionally and then it calls fork() and it creates lots of child processes
- the child processes are copies of the main process
- a pre-forking web server(puma cluster mode) boots a single 'parent' process, boots the application, listens on the socket, then calls 'fork(2' to create child processes.
- there's no communication between processes unless we set it up explicitly. they don't share memory. they are not sharing anything with the main except they do share the socket
- when main process opens up socket and oh we're gonna do this, that socket object is passed to the child process as a copy and so they're both listening on the same socket at the same time
- the child process will listen on the socket and call accept ant that accept call is, it says i want to pick something off of the socket, there's a request there, please give it to me.
- after the app boots, child processes serve requests. the parent does not.
- the parent processes's job is to receive signals, and to boot new child processes.
- **when you say `puma -w 4`, we create one master process and then create 4 additional worker processes**

![](/assets/mental-model2.png)

- we've got puma parent process sitting off on the side somewhere, it's not actually listening to the socket or anything
- we've got a socket with several puma child processes listening to it.
- and each of those child processes indivisually not sharing anything, they are turning http into rack and then calling the rack application with an environment hash

## threadpool

- puma, inside of every process, each process has one thread pool
- and that thread pool can be from zero to however many threads you tell us and that thread pool is what actually calls the application
- so we'll create, let say 5 threads and each of those threads will be picking work up and calling `app.call(env)`

- we've gotta work array like works we can do, we call this to do internally in puma
- the thread pools just like a bunch of threads that tries to consume work and then when you add work to the to-do set, that thread automatically picks it up because it says oh there's work on the to-do set i'll grab it now 
```ruby
work_to_do = []
thread_pool = Array.new(5) { Thread.new { block_that_consumes(work_to_do) } }
# ...
workd_to_do << Request.new(this_request)
```

- the Global VM Lock(GVL) allows only one thread to run Ruby code at any pointt in time
- GVL is like a special machine being passed around between customs agents at a border checkpoint
- the main thing ruby processes do other than run Ruby is to wait on I/O
- for example, when you make a db query, there's a certain amount of time that you're spending, just waiting for sockets. that time is spent not actually in Ruby and that GVL is released during that time

- since Ruby 3.0, the GVL is no longer exists
- now there's one VM lock per ractor. the Ractor VM Lock allows only one thread in a Ractor to run Ruby code at any point in time.
- puma doesn't use Ractors yet

- a puma process running a 50%-of-time-in I/O wait app, in MRI, with about 4 threads can process 2x as many requests as it could with one thread

- this is Puma's primary benefit: more throughput for a similar amount of memory resources.

![](/assets/mental-model3.png)

## getting into real internals of puma
- the Reactor's job is to buffer requests, so that only complete reqeusts are sent to the thread pool
- this prevents slow clients(uploads, attackers) from consuming our ThreadPool
- the problem with that thread pool design in the previous slide is what if somebody decides to upload a 20mb file to my server and they are uploading it on their 3g phone. if i just handed reqeusts to the thread pool that would mean that thread would be sitting there waiting for that 20mb file to upload and it would take a really long time and i'd lose 20% of my capacity
- Unicorn lacks a Reactor, which is why you should always use a proxy in front of it, like nginx

![](/assets/mental-model4.png)

- in between the thread pool and the socket, we've got this reactor whose job is to run this event loop and take raw socket data turn it into HTTP and then the ruby code turns that into a rack env.

























