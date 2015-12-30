---
title: Performance of Prototyping in Javascript
---

The post upnext might be obvious, we had our "DUH"-moments ourselves too, but here is something that can keep you from a lot of performance-issues...

##Performance issues???
Imagine you're writing code where you have a lot of things, each represented by a javascript object, all having the same behaviour but with different property-values.
Of course you should make a class-object of it, and instantiate it over and over again.

There are multiple ways to do this, but we recently discovered that *how* you do this, can make a **HUGE** difference. Just to mention, at neoScores, we reduced the time to calculate a full-blown orchestral score (comparable with a Mahler Symphony) from **10 seconds per page**, to only **60 milliseconds(!!!) per page**.

##10 seconds vs 60 milliseconds!? What did we do (wrong)?
We used to make a class-object, with a lot of properties and methods, like this:
{% highlight js %}
// Define our Thingy CLASS-Object
function Thingy(props) {
  var i = 0;
  this.add = function(value) {
    i += value;
  }
  // lots and lots more methods
  // ...
}
{% endhighlight %}
And we Instantiate one new Thingy-Object for each of our requested thingies
{% highlight js %}
for(var i=0; i<allOurThingies.length; i++) 
  new Thingy(thingies[i]);
{% endhighlight %}

Easy..., yes?
Well... Imagine we have a million thingies and we measure the time to run through our *"for-loop"*. That actually does take some time.
Let's think a moment about this. Every instance of each one of our thingies, allocates some space available in our memory, so it can store all variables AND methods to run. Keep in mind that every method you declared inside your Class Object, also needs to allocate some memory space.

##So, how are we doing things now?
We only need to define a Class Object with the bare minimum we need
{% highlight js %}
// Define our minimal Thingy CLASS-Object
function Thingy(props) {
  this._i = 0;  // make sure we already preserve the memory, needed for an integer, for this property
}
{% endhighlight %}
And we add our methods, but now we use **prototyping**
{% highlight js %}
Thingy.prototype.add = function(value) {
  this._i += value;
}
{% endhighlight %}

Instantiate *that* for a million times and you'll see we have a very, VERY, much cheaper script than we used to have. Check this [jsperf](http://jsperf.com/prototyping-classobjects)
So, yes, we needed to "refactor" all our code to remove the inner methods of each of our Class Objects, and transform these methods into prototype-functions. FYI: this took us more or less just one week.

##Any caveats?

Yes... There are caveats. Like Douglas Crockford likes to call it in a beautifull wording: *"Privileged Functions"* ... are not possible using this simple implementation of prototyping. "Privileged functions" is the reason why we used to work with the first method I described, we wanted *private vars* (or as close as we can get to this in the javascript-world). 
When you instantiate a function in javascript, it gets it's own new "scope" which means that a new variable in this new scope will not be known outside of this scope. That's why you can call it a "private var".
{% highlight js %}
function Thingy(props) {
  var x = 'hello private world';
  this.y = 'hello public world';
}
Thingy.prototype.speak = function() {
  console.log(x); // -> undefined
  console.log(this.y); // -> 'hello public world'
}
var thing = new Thingy();
thing.speak();
// -> undefined
// -> 'hello public world'
{% endhighlight %}

This all gets realy interesting when you start combining this with getters and setters!!!
You say what!? That's for my next blog...

Meanwhile, enjoy reading:  
[http://javascript.crockford.com/private.html](http://javascript.crockford.com/private.html)
[http://javascript.crockford.com/prototypal.html](http://javascript.crockford.com/prototypal.html)
