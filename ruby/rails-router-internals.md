https://www.youtube.com/watch?v=lEC-QoZeBkM

# RailsConf 2018: Re-graphing The Mental Model of The Rails Router by Vaidehi Joshi



# how rails router work under the hood
- the router basically allows us to recognize URLs.

![](/assets/router1.png)

- the router knows where to send you.

the first step to investigating how the router works is, of course, by identifying exactly where it is.
you have to figure out where the router comes into plays in terms of the life cycle of your apps

```
-> bin/rails middleware

use Webpacker::DevServerProxy
use Rack::MiniProfiler
use ActionDispatch::HostAuthorization
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
... 
use Rack::ConditionalGet
use Rack::ETag
use Rack::TempfileReaper
use Warden::Manager
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use Bullet::Rack
use OmniAuth::Builder
use OmniAuth::Strategies::Facebook
use OmniAuth::Strategies::Naver
run Lookingood::Application.routes 
```

all they really are just the middleware in the order that is's executed in your app.

what is middleware again?
it's a rack app that takes another rack app as an argument
you can have rack app that are standalone which means that they aren't initialized with other rack apps
rack app that are standalone are often referred to as rack endpoints.

## what is rack?

controller actions are just rack endpoints.

```ruby
Rails.application.routes.draw do 
	# root 'welcome#home'
	root to: proc { [200, {}, ['omg it works!']] }
end
```

```
Application.routes 
```

since this is the end of the stack, we need to figure out what this `.routes` thing actually returns.

`lib/rails/engine.rb` it has a definition for a routes method.
```ruby
def routes
	# ...
	@routes
end
```
this routes method returns an instance of all of our routes.

now we know that this is how the request actually gets through the middleware stack into the router.

the question is how does the router route it?

```ruby
get 'recipes/:id', to: 'recipes@show'
post 'articles(.:format)', to: 'articles#create'

# more routes
```
how does the request that comes into a router get sent to the right one of these?

the naive solution that, maybe even on a first glance, it make sense to just work our way through all of thr routes in our app until we find right one.

we could maybe even just iterate through, and write a loop, and we could use a regular expression, and just check the request, and see, 'does it match this route?, does it match this route?'
```ruby
if request_path =~ /^\articles$/
	# go to articles#index
elsif request_path =~ /^\recipes$/
	# go to recipes#index
elsif ...
	...
else
	...
end
```

it's not the best because you might have a lot of routes and your `if` statement's going to get very long.
another issue is that, as your routes file grows, this is not gonna scale that well.
let's say we have n number of routes. this is actually going to run in linear time,O(n), because in the worst case, you could be looking for a route that doesn't even exist, and you gonna check through every single route in your route file, and end up empty-handed.

you'll start to realize how painful this initial solution really is when you start to compare it back to that post office example.
imagine that the post office just looked through a long list of addresses envery single time it got an envelope, and it would just check to see if the addresss on the envelope matched the one that it was looking at. as you can imagine, that's a very long list, it's not efficient, the post office is already pretty slow getting my mail to me.

![](/assets/router2.png)

what we really want is to just narrow down which routes we're looking at.
instead of iterating through our whole list of addresses, instead of looking through all of our routes, what we really want is to narrow it down so that we're not spending time searching for things that are definitely not gonna be a match.

there's definitely room for improvement in our strategy here.

Journey is routing engine that, it used to be standalone library, and it was merged into Rails around Rails 4.

## where does journey fit in?

where does Journey fit into the router?

remember when we called that method `routes`?
that routes method returns a routes instance, which we can see, it creates something called a routes set. this is like, we're actually getting to our set of routes. we're finally inside of a router.
```ruby
@routes ||= ActionDispatch::Routing::RouteSet.new
```

what is the routes set?

if we look at the definition of route_set, and now we're inside of action_dispatch, actually, we'll see that a set has Journey.Routes within it.

```ruby
# action_dispatch/routing/route_set.rb
@set ||= Journey.Routes.new
```

this is the first time we touch Journey's code. At this point, we're gonna actually take a break from Journey's code itself, and we're gonna focus on the general concepts of what the engine does. that way, if, after this talk, you want to go back and look at the code, you'll be well-equipped to understand what it's doing and how it's working and you'll know where to start too.

Journey uses a computer science concept called a graph in order to make life much easier for our routes.
graphs are kind of like a data structure.

all you really need to create a graph is one node, but usually, you're gonna see graphs with multiple nodes. those nodes are gonna connect to each other with something called edges or links.

![](/assets/router3.png)

edges can be either undirected or directed, and for the purpose and context of Journey today, we're really only gonna be dealing with directed graphs.

let's go back to our post office for a second.
this post office is gonna hang around, but hopefully, you'll see that the metaphor kind of continues, and it helps make what Journey's doing a little bit easier to understand.(이부분은 걍 주저리주저리하는거인듯)

if you think about how a post office works ideally, no post office is just randomly putting in envelops into whatever mailbox. there's some sort of system and order to how it functions. the system that it uses is very efficient in solving a problem that would otherwise not scale well. 

![](/assets/router4.png)

specifically, in a post office, the rules, which is really the algorithm that the post office abides by, for figuring out where to route your mail is by starting with the most broad part of the address first. then we narrow down from country, zip code, then state, and city and street and then your actual house address.
there's a mechanicalness to how this works, and it's kind of an algorithm in and of itself.

the same is true with the rails router.
the idea of narrowing down and matching an address to help avoid unnecessary searching is the same idea that you can find within Journey.

Journey uses a graph data structure to help the router figure out how and where exactly to direct a request by matching against the URL, which is the address of the request when it comes in.

![](/assets/router5.png)

we can imagine, when Journey looks at a request, the first thing it has to do is figure out where on earth this request actually needs to go in the routes file.

```ruby
Rails.application.routes.draw do 
	root 'welcome#home'
	
	resources :articles
	resources :recipes
	resources :comments
end
```
that's how it needs to start its search.
for example, when we have a request that comes in, like `/recipes/:id`, we have to start looking for a specific recipe.
what we eventually want to do is send it to the controller where we can look for a specific recipe by id.
but before we even can find the controller, it seems silly to even consider the root, or articles, or any of the comments routes, because we know for a fact, it's not even gonna be there at all. 
you and i know this intuitively, because we can just look at the routes file and be like 'this seems obvious.'
of course, this becomes even more important because every routes file where we have a resource, we're actually narrowing down and eliminating a huge number of routes that we don't have to search through.

![](/assets/router6.png)

what if, instead of looking at every single route, like our original, naive implementation, what if, instead of that, we did something similar to what the post office does?

![](/assets/router7.png)
![](/assets/router8.png)

what we really need to do is smartly look at the address when it comes in, and narrow down the country, state, city, steet address of our request. and then we can send the request along. 
but first thing first, when a reqeust comes in, it needs to be read. we can't really worry about sending it to the right address if we can't read the address. we need to make sense of what's on that request envelope.

## how does the router read?
how does the router read? that seems like a very big skill for a router to have. well, we have to teach it.
it turns out that Journey reads similarly to how you and i would read. in fact, the way that Journey reads is exactly how compilers will read code too.

```
the cow jumped over the moon. 
```
when you read a sentence like above, what your mind is really just doing is looking at this string of characters, and you're looking at the capitalization, the punctuation, the spaces, and in your mind, you're dividing it up into coherent letters.
your mind takes those letters and builds it into a sentence that follows some sort of grammar so that this sentence has some meaning to you. the ability to break up this string into pieces that make sense, and string it to follow a grammar is basically what Journey needs to be able to do. it's replicating what you and i can do as humans so easily. if a human can do it, then obviously, there's gotta be a way to a computer to do it too. 

![](/assets/router9.png)

this is what Journey does with the request address that's on every single request that comes in, the URL string. except, of course, in order for Journey to read this, it also has to know what a letter is, and it needs to know what grammer is, and then it needs to be able to group them together so that it makes sense of what on earth is on this request envelope.
Journey has to perform a process called tokenization.

## tokenization
tokenization seems hard maybe, but all it really is the work of breaking one expression down into its minimally significant parts. there's a name for those little parts as well, and those individual pieces are called tokens.

![](/assets/router10.png)

thankfully, Journey does have some help when it comes to tokenization. there's one program within Journey that handles the work of tokenizing. technically speaking, it's not exactly a program, it's a class, it's called the Journey scanner.
the Journey scanner actually inherits from Ruby's StringScanner class.
this scanner can take any string, and it's gonna follow a set of defined rules in order to derive tokens, which are the minimally significant little bits of the expression. and it's going to do that from whatever input that we give it. the input we're gonna give it is a request URL.

![](/assets/router11.png)

we can actually see the scanner in action by going into the rails console, and creating a new instance of this. we can also see it do the work of tokenizing a string into individual tokens too.

![](/assets/router12.png)

if you call next_token on the scanner instance, you can see it actually splitting apart our request URL, `/recipes/:id`, and you can see it split that request string into individual expressions, or tokens.
Journey's scanner recognizes a handful of tokens, including slashes, string literals, left and right parentheses, amoung a few others.
you can imagine that it would be important for Journey to be able to distinguish, hey these are the letters, and the words that make up my string that i'm trying to parse.
the scanners helped us out. it knows how to divide up a string into words or tokens, 

that's only half the battle because words mean absolutely nothing unless there's some sort of grammatical rule to follow. the next step is for us to figure out what's going on with thos words, and make some sense of them.

Journey is just like us. they have to follow grammar too. what you and i are intuitively able to do, Journey needs to learn how to do, and the way that Journey actually solves this probelm is with some help from another class called the parser. 
the parser's job is to take those tokenized pieces, the tokenized string, and make some sense of it. the parser does this by utilizing yet another computer sicence concept called a syntax tree.

## syntax tree
now, syntax trees might sound like very hard and based in linguistics, which they actually are, but we're not gonna have to worry about that today. we'll just cover why they're important in this context.

you might remember, in elementary school, you probably, at some point, had to diagram sentences. and you had to distinguish different parts of the sentence to understand and learn grammar, whatever your first language might be.

![](/assets/router13.png)

you probably had to do something like this, where you took a sentence, and you turned it into a tree of the words in the sentence in order to figure out the parts of the sentence and how to structure it.

![](/assets/router14.png)

that's just what a syntax tree is. it's an illustrated version of the structure of a sentence. it turns out, it started out in linguistics, and then it moved on to education, and educators now use it to teach grammar to kids. as it turns out, the parser creates a similar kind of syntax tree for Journey, and help it make sense of the grammar of its language.

![](/assets/router15.png)

here's an example of syntax tree that Journey would generate for a route that corresponds to the show action for a recipes controller.
you'll notice that, if you read the tree from the bottom, and start to work your way up the nodes, and start to reduce the nodes as you work your way up, the combined expressions actually start to reduce until all you have left is a single expression at the end. 
in fact, for every single route that we define in our routes.rb file Journey's parser is gonna create a syntax tree for it.

if you are curious, you can actually see this in action in the rails console. you can output the syntax tree for a request, and it'll turn into an HTML string and you can actually use graphviz to visualize it.

![](/assets/router16.png)

## intermediate step
now you have a bunch of trees, but it still doesn't really help us combine all of our routes together.
these are kind of like disjointed parts in the post office. we need one single system to solve this problem.
what do we do with all these syntax trees? well, you might remember our graph. this is where that comes into play.
Journey uses a graph to route your request, and that graph is made up of those systax trees.

![](/assets/router17.png)

Journey builds a graph of all your possible routes by combining these syntax trees, and Journey's code actually refers to this as something called a generalized transition graph, or GTG.
but our purposes, all you really need to do is think of it in the context of a state machine.

![](/assets/router18.png)

the state that we're really dealing with is the request URL that we're parsing, and how far along the URL we've parsed.

when a request makes its way to a router, Journey's going to walk this graph, one node, or one state, at a time to match a subset of the request URL.

![](/assets/router19.png)

basically, when it reads a request URL, one section at a time, it's going to walk down the graph and based on which nodes match the state it's currently in, it'll progress to the next part of the graph. if a node in the graph, which is one node of many syntax trees that have been combined, corresponds, that's how Journey knows to walk down the next path inside of the graph, or rather within the graph. it will keep walking down this tree until it finishes reading the address, until it finishes parsing the request URL string. and if it finds a node that matches the state of that string at that moment when it finishes reading, well, that's how it knows exactly which route that the string that came in with the request corresponds to.

![](/assets/router20.png)

so if you really think about it, you and i probably could've done this super easly if we had just looked at the routes file and been like 'yeah obviously we want to go here', but what we can do in one glance and identify very easily, it's so simple for us to find the controlller and action that correspond to this, Journey has to do all this work in order to mimic and replicate what we're doing. but it does it very fast, so it's ok. in order for Journey to do what we can do, it has to implement data structure under the hood.

in fact, what you can really imagine is that there's a little robot that's walking down the path with this request URL similar to how there's probably someone in a post office who is like, 'all right, i'm gonna sort this in the correct way'
we can imagine this little robot walking down this tree for us, walking through our state machine for us. and it's systematically checking the request URL string, and deciding which path that it should take in our state machine graph structure.

another name for this same concept that i just described here is called 'nondeterministic finite automaton, or NFA'.
an NFA is just a type of state machine, and it handles a little bit of input at a time, and decides, 'how am i gonna proceed and transition forward? this it the current input that i have, this is the current state. i'm gonna make my choices based on what i have now, and where i should go forward in the graph next.'

## two possible outcomes
there are basically two outcomes for our little robot who's walking down our graph for us. there's only two things that can ever happen.
wither we'll find something that matches the state, the string request URL that we took in as our input, or we don't. 
when we get to the end of our string, and we're at a point in the graph that actully works, that corresponds to a real route, then we know that there's a route that matches our request URL. and if we're able to find a route that matches, and that means we found an acceptable place, an acceptable controller and action to send the request on forward to. this is also referred to as an accepted state.
if there is an acceptable route to be found, then all Journey have to do(or the router has to do) is just dispatch the request to corresponding controller action.
so far, we haven't looked at any real Journey code, but i think we're finally ready to make it happen. this is actually not technically code. this is the visualizer.

https://tenderlove.github.io/fsmjs/

![](/assets/router21.png)

it just kind of looks like this. you'll notice, it doesn't look that different from that colorful graph i had before. it's the same idea, and same concepts, and hopefully, looking at it now, you can kind of see some sort of similarities there.

here's an example of what that same NFA would look like if we were searching for a valid route. this is an example with just four routes, so you can imagine how big it would be in other cases.

![](/assets/router21.png)

so if Journey's able to find a vaild accepting state, and it can dispatch the request to an actual controller and action, then it will go ahead and do that.

what happens if there's a request URL that's garbage, or is typo, or just isn't a real URL? what happens to our NFA then?

### invalid, unacceptable route
in that case, you have an invalid unacceptable route, the term for this is a rejected state. you really have no choice but to leave the state machine, because there's nowhere you can go, and probably it would make sense to raise an error, or indicate somehow that, 'hey, this is not a real route'. this is what the NFA will look like in a rejected state.

![](/assets/router22.png)

you'll notice that, when you reject out of the router, if you've ever run into issues routing with your rails app, you might've run into an error like this.

![](/assets/router23.png)

in fact, Journey is the starting point for where your rails app knows, 'hey no route matches'. the error begins from this point.

as it turns out, you can simulate the same equivalent NFA that i just showed you there in your own Rails app. there is a visualizer that you can run, `Rails.application.routes.router.visualizer`(작동안한다), and you can actually see your own internal NFA.


## well this has been quite a 'journey'

now that we've made it through entire life cycle of the request making it into the router, hopefully you see that it's actually not as magical as maybe you initially thought.
in the process of trying to understand the internals of the Rails router, we actually learned a lot.
namely, Journey is a regular expression engine, but we learned that it uses tokenization, it uses scanning, it uses trees and grapghs, and automatons, all under the hood.
these concepts are not unique to Journey. you can actually find them in lots of places, including within languages, other frameworks, other parts of Rails, and in compilers.















