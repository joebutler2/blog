---
layout: post
title:  "What is a Closure?"
categories: article
tags: 
date: 2011-03-19
---

You can see closures in different web programming languages such as Ruby, Javascript, ActionScript 3.0 and newer versions of PHP. It is a powerful tool and can be an elegant solution given the right circumstances. With that background in mind, let's give it a specific definition.

> A closure is a function that encapsulates a scope and is able to be passed around as an argument.

This enables you to define a function that will "close over" variables in the current scope, and then later execute that function in a different context. It may sound complex, but a simple Ruby example should clarify.

{% highlight ruby %}
intro = 'Hello '
say = Proc.new { |name| intro + name }
puts say.call('Tim') # => Hello Tim
{% endhighlight %}

First we define a local variable, named intro, which is used in the closure on line 2. Breaking down line 2 we see that a new Proc object is created and stored in the "say" variable.

> A Proc is one of constructs Ruby gives us to create a closure. Note that Ruby gives us several other constructs for doing this, but we'll save that for another article.

Taking a closer look at line 2 you can see a block variable called "name" (it is wrapped between 2 vertical pipes). This is a parameter that you will pass to the closure when you want to execute it like line 3 in our example. If you run the example you will see that it outputs "Hello Tim".

While a simple example helps give us a basic understanding, we should demonstrate the ability to execute the closure in a different context. Continuing with our previous example.

{% highlight ruby %}
class Greeter
  def greet(closure)
    closure.call('from Greeter')
  end
end
puts Greeter.new.greet(say) # => 'Hello from Greeter'
{% endhighlight %}
    
We define a Greeter class with an instance method of greet that accepts a closure as an argument. The greet method then calls the closure passing it the string 'from Greeter'. It is important to realize that when we do this we are injecting data from a new context than what the closure was defined in. This is one reason closures are powerful, they can combine pieces of data from different parts of the system. Be careful though since this can increase coupling in a system.

Ruby provides us with constructs, such as Proc objects, for creating closures. As for Javascript, we use anonymous functions. The following is the Javascript equivalent of the previous Ruby examples.

{% highlight javascript %}
var intro = 'Hello ';
function say (name) { 
  return (function (name) {
    return intro + name;
  })(name);
}
console.log( say('Tim') ); // Hello Tim
    
function Greeter () {
  this.greet = function (closure) {
    return closure('from Greeter');
  }
}
var greeter = new Greeter();
console.log( greeter.greet(say) ); // Hello from Greeter
{% endhighlight %}
    
There are two very noticeable differences between the Javascript and Ruby examples. The first one is the self executing anonymous function that is returned on line 3. The anonymous function is immediately invoked by the parentheses on line 5. This will return the result of the anonymous function rather than the function itself (more about that soon). On line 5 we are also passing an argument into the function, this may seem a little tricky since we are passing the "name" parameter that was passed into the say function.

The other big difference between Javascript and Ruby is how functions are called. In Ruby we can omit the parentheses on a function call and it will still be executed. Javascript, on the other hand, will return the function itself if it is called without parentheses. To quickly demonstrate.
  
{% highlight javascript %}
function sayHello () {
  return 'Hello';
}
console.log( sayHello ); // function definition
console.log( sayHello() ); // Hello
{% endhighlight %}
    
This idiosyncrasy is the reason why we made the returned function self-invoking, so that we didn't have to execute it on a separate line just get the closure. Hopefully this gives you a good sense of what closures are, now let's see how they're used in practice.

Use cases for closures
----------------------

We'll review some common scenarios that closures are used for in Ruby as well as Javascript. Starting off with the ubiquitous link_to function from the Rails UrlHelper module.

{% highlight erb %}
<%= link_to search_path do %>
  <span>Search</span>
<% end %>
    
#=> <a href="/search"><span>Search</span></a>
{% endhighlight %}
    
Using a closure we are able to create an elegant API that is easy to read. The erb looks almost like the output. Consider the alternative without a closure.

{% highlight erb %}
<%= link_to search_path, :inner_html => '<span>Search</span>' %>
{% endhighlight %}

Okay that isn't too bad, we lost the visual representation of the erb wrapping the inner markup. What about a link that has richer styling, and a more complex path?

{% highlight erb %}
<%= link_to search_path(:key_1 => "value_1", :key_2 => "value_2"),
      :inner_html => '<span>Search</span>',
      :class => 'btn',
      :id => 'search' %>
{% endhighlight %}

Even less readable, the inner_html argument is now being lost in the noise. Hopefully this clearly shows the readability benefits of using closures.

While this demonstrates a closure used in an API we'll get a better understanding if we implement our own.

### The Javascript Module Pattern

Javascript is a Prototype based language and has a different approach to objects than in a more traditional object-oriented language. One of these "missing" features is access modifiers for declaring members of an object as private or public. In Javascript everything is public which means the language doesn't have a way to enforce information hiding. Perfect opportunity for closures to work around this inherent "weakness".

{% highlight javascript %}
var Person = (function () { 
  // Private vars
  var name,
      age = 20,
      ssn;
      
  // Private methods
  function createSSN () {
    var uniq = (new Date()).getTime();
    uniq = uniq.toString();
    ssn = uniq.substr(uniq.length - 11, uniq.length - 1);
    ssn = Number(ssn);
  }
  createSSN();
  
  // Public methods
  return function () {
    this.getName = function () {
      return name;
    }
    this.setName = function (newName) {
      name = newName;
    }
    this.getAge = function () {
      return age;
    }
    this.getSSN = function () {
      return ssn;
    }
  }
})();

var john = new Person();
john.setName('john');
console.log(john.getName()); // john
console.log(john.getAge()); // 20
console.log(john.getSSN()); // unique 10 digit number
{% endhighlight %}

Nothing exactly new here, it is just a creative use of the techniques we've already looked at. We set the Person variable to a anonymous self-invoking function. Inside of this function we have our private member declarations, this includes variables and a createSSN method. Once the Person object is created these members fall out of scope and are only accessible through the closure. Now we get to the meat of the module pattern, the closure itself. It turns out the anonymous function that is assigned to Person actually returns a function, this is a closure just as we've seen before. Since this is a closure it encapsulates the lexical scope and can be returned, effectively giving us private variables and methods.

### Deferred execution

Another great aspect of closures is that we can define some behavior and then execute it at a later time when needed. This commonly seen in Ruby and Javascript, sometimes in the form of callbacks. The popular testing library RSpec provides a before method for sharing data between examples.

{% highlight ruby %}
describe Cli do
  before :each do
    @output = double('output').stub(:puts)
  end

  it "sends a welcome message" do
    @output.should_receive(:puts).with('Welcome User.')
  end
  it "prompts for first command" do
    @output.should_receive(:puts).with('Enter command:')
  end
end
{% endhighlight %}
    
Even if you're unfamiliar with RSpec and BDD you should be able to get something out of this example. The first line opens up the ExampleGroup, or TestCase in TDD terms. Then you see the before method, it takes a block and runs it before each of the following examples. In this case it is going to set up a dummy data object and assign it to the @output instance variable. The next two method calls then use the @output variable in their assertions. This use of closures can made your code more DRY and easier to understand.

Another cool example of deferred execution is implemented directly in the Ruby core, at_exit. This method takes a block and calls it once the program completes or throws an error.

{% highlight ruby %}
at_exit { puts 'Goodbye' }
{% endhighlight %}
    
Running that command will simply display "Goodbye", but this humble method can be quite useful. It can ensure certain resources are cleaned up before exiting. Or you can use it to write logs files. Even something as simple as a salutation to the user in a command line interface.

Not only is deferred execution common in Ruby but also Javascript. You usually see them in the form of callbacks (a function that is called in response to an event). Since Javascript is designed for the purpose of handling user interactions, finding an example is trivial. So we'll try to make this interesting by defining a callback in a loop.

{% highlight javascript %}
var data = ['item 1', 'item 2', 'item 3'];
for (var i = 0; i < data.length; i++) {
  var logElement = (function(item) {
    return function() {
        console.log(item);
    }
  })(data[i]);
  setTimeout(logElement, 700 * i);
};
{% endhighlight %}
    
In this situation we're only storing the closure in a variable to increase readability. It would work fine if we passed the self-invoking anonymous function directly as the first argument to setTimeout. Since data[i] only exists inside of the loop we have to pass it into the outer function so it will be stored inside of the closure. Once the time limit is hit it will then call our closure, printing the data value to the console.

Closures in other Languages
---------------------------

Earlier we said that newer versions of PHP support closures. This feature was added in version 5.3. At the time of writing, PHP 5.3 is still fairly new, so be sure to check your hosting environment before utilizing them in production code. For more information be sure to check out the [official PHP documentation](http://PHP.net/manual/en/functions.anonymous.PHP) and this [article from code utopia](http://codeutopia.net/blog/2009/02/20/closures-coming-in-PHP-53-and-thats-a-good-thing/).

Another web programming language that features closures is ActionScript 3. Both ActionScript 3 and Javascript are based on a set of standards known as ECMAscript. In a sense they are almost like language cousins. Containing similar syntax and having other commonalities, including how they handle function calls [as mentioned earlier](#js-function-call). Despite their homogeneity, there are still differences. ActionScript 3 is a more traditional object-oriented language while Javascript follows a Prototype based path. While these Javascript examples won't translate directly, they should give you enough to start experimenting. One caveat, be careful of the different ways scope is handled in Javascript vs ActionScript 3.

In conclusion
-------------
While at first closures may have sounded like a esoteric computer science concept, I hope this article has taught you how to practically implement a closure. They can greatly simplify APIs and make the code easier to read and maintain. This translates into saved time and money for clients, and hopefully makes for happier developers.

Don't forget that these techniques can lead to tightly coupled code. If a closure is exposing an object from another part of the system, changing that object's interface could easily break functionality. And if that closure was passed around and called in different locations you could have a nightmare updating that codebase.

Now that you are aware of the potential dangers you are more likely to avoid them and use closures to their fullest.

If you have any useful closure examples, or questions please join the discussion.

Further Reading on Closures in Ruby and Javascript
--------------------------------------------------
* [Interview with Yukihiro Matsumoto about Ruby Closures](http://www.artima.com/intv/closures.html)
* [Original Javascript "Module Pattern" Article](http://yuiblog.com/blog/2007/06/12/module-pattern/)
* [Javascript Closure Use Cases](http://msdn.microsoft.com/en-us/scriptjunkie/ff696765)