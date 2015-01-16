---
layout: post
title: "A Swift Corner Case"
date: 2014-11-21 01:10:00 -0600
---

At our local NSCoder meetups, a large percentage of the attendees are students. They're learning iOS programming for the first time in a class taught at our local university by my good friend [Tyten](http://twitter.com/tyten).

<figure>
	<img src="/images/tiger-hair.png" alt="A screenshot of an Apple event with the University of Missouri's Swift class called out.">
	<figcaption>The class was mentioned at Apple's iPad launch event in November.</figcaption>
</figure>

Last night, one of the students had a rather odd error appearing, and it took us a while to fix and even longer to figure out the cause of. The cause ended up being the combination of two somewhat-obscure Swift behaviors, so it was pretty fun to figure out.

The student had put together a fairly standard delegate pattern: he had a protocol he created, and a class that implemented it. The class that called the method did so when it was finished fetching and parsing data from CloudKit. However, the CloudKit methods are called on a background thread and the delegate method interacted with the interface. To fix this, he wrapped the call to the delegate method in a block run on the main queue. The code ended up looking something like this:

{% highlight swift %}
import Foundation

protocol FooDelegate {
	func didUpdateFoo()
}

var delegate :FooDelegate?

//elsewhere…

NSOperationQueue.mainQueue().addOperationWithBlock {
	delegate?.didUpdateFoo()
}
{% endhighlight %}

Go ahead, try running that code in the Swift REPL. You'll get an error that looks like this:

```
error: cannot convert the expression's type '() -> () -> $T3' to type '()'
```

Well… that's not very helpful. And this is the error that we were seeing in Xcode. After a bit of trial and error, we managed to get it to work by wrapping the method call in an `if let` block, like so:

{% highlight swift %}
NSOperationQueue.mainQueue().addOperationWithBlock {
	if let aDelegate = delegate {
		aDelegate.didUpdateFoo()
	}
}
{% endhighlight %}

So what caused this bizarre error? After getting it working, I went back to the [Swift book](https://itun.es/us/jEUH0.l) to try and figure it out. Because of the prominence of the chained `delegate?` call, that's where I started. Any call to a method or property that uses optional chaining will return the same type as it normally does, but wrapped in a optional. Which makes sense&mdash;any broken link in the chain will cause the whole thing to return `nil`. This is true *even* for methods that return `Void`. Chained calls will return `Void?`.

> “If you call this method on an optional value with optional chaining, the method’s return type will be `Void?`, not `Void`, because return values are always of an optional type when called through optional chaining.”

But… that shouldn't have caused any issues in our program, should it? We weren't doing anything with the return value of that method call.

Not so. It turns out that Swift's closures have another trick up their sleeve: implicit return.

> “Single-expression closures can implicitly return the result of their single expression by omitting the return keyword from their declaration […]”

Because of the combination of optional chaining and implicit return, we suddenly were left with a closure that was automatically returning something of type `Void?`, which clashed with `addOperationWithBlock()`'s expectation of `Void`. In the end, the trial-and-error fix we implemented fixed *both* cases. We weren't chaining any more, so `didUpdateFoo()`'s type was `Void`, not `Void?`, and our closure was now more than one expression, meaning it wasn't implicitly returning.

This was an interesting bug to diagnose&mdash;it led to some interesting parts of Swift I had missed so far. Implicit returns was something I'd completely missed, and I'm curious to see how it might be used better.
