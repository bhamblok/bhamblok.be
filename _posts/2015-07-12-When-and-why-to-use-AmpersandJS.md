---
title: When and why to use Ampersand.js
---

Building our project neoScores, we've tackled a lot of problems in search for the ideal framework. The more frameworks we tried, the more often we started over again.

In the beginning we were convinced that a full-featured CMS-framework could serve all our needs. Several months later we concluded that the only thing we were actually using of this open-sourced bulky system, was the user-management part of it. Everything else was custom build on top of this framework and ... unfortunately ... we had *"vendor-locked"* ourselves.

This blog talks about the bumpy road we've taken to end up with something we realy love: [Ampersand.js](http://ampersandjs.com/).

## What do we actually need from a framework?
We are working on a Web App which feels like a native app. Our app needs to run offline, so it would be nice if it runs like a single-page website, but then again, we need to show multiple different pages or views, preferably via a path change in the url. Further on, content needs to interact with user's input or should dynamically be generated via data coming from Object-models.

> ### 1. Multiple pages
The absolute core-task of our app is to open up different pages, or as it's called in the web-app-world: __*views*__. Because our app runs like a single-page app, we need some kind of a **router** which navigates to different views depending on changes of the path in the url.

> ### 2. Models and Collections
Our app needs to show some stuff to user. In our case [neoScores](https://app.neoscores.com), we need to show a musical score, a piece of Sheet Music. A musical score is a combination of a title, composer, copyrights, (...), and of course a bunch of staffs, notes, bars, accidentals and a lot, lot more. So we need to be able to bundle all these properties into something we can manage easily, while being flexible to change some of the properties via some user input tools.
All of these properties together are the construction-blocks of our **"Model"**.

> On the other hand, we need to be able to list up all the scores for one user into his "user's library". The user needs to be able to make subsections of his library (like "Playlist" or "Setlists") or he needs to be able to change some properties of all his scores together at once via some user input tools. A **"Collection"** is exactly what can do all of this.

> ### 3. Offline Use
Our app is build on top of the concept off **"Offline First"** which means that LOCAL (in-browser) databases (*"indexDB"*,*"WebSQL"*,*"LocalStorage"*) must be addressed to look for the requested data, in combination with our ONLINE databases to synchronise everything with the cloud, in real time, over multiple accounts, devices and platforms.

## Object Binding
From the technical point of view, we should be able to create Models (Objects) and Collections (Arrays of Objects) which bind to the UI (the DOM) automatically. The concept of *"Object Binding"* is exactly what does all of this. It would be realy ... realy ... convenient when our code simply updates the screen, just when a value of a property of one of our models has been changed.

So, when somewhere in our code, the value of the title of our score is changed ...
{% highlight js %}
score.title = 'My new title';
{% endhighlight %}
... the screen (our view) updates itselves with exactly *that* string that our variable (score.title) just has been given.

To do just that, a lot needs to be done:
- you (as the developer) decide that at some point the value of a variable needs to be changed to 'My new title'. (eg. when the database-request gives a response whith some new data)
- an event needs to be triggerd to tell your code "HEY, YOUR VARIABLE (score.title) HAS BEEN CHANGED"
- some other code needs to be listening to this "change-event"
- when this "other code" picks up this event-trigger which says *something has been changed*, it needs to update the browsers *DOM* with the new value.

## Getters and Setters
Look closely at the code-example above... it just says:
{% highlight js %}
// example for an Ampersand-Model
score.title = 'My new title';
{% endhighlight %}
THAT'S IT!!!! You don't need to learn some new syntax invented by the creator of some framework. So ..., how the hell are all the events I just talked about invoked???

Ampersand.js does that for you using javascript getters and setters.
When you (being the developer) define your model (our score)...
{% highlight js %}
var Score = Model.extend({
  props: {
    title: 'string'
  }
});
{% endhighlight %}
... Ampersand.js will make a getter and a setter from the property 'title'.
Basically as follows:
{% highlight js %}
// dummy-code
function constructionOfMyScoreModel() {
  Object.defineProperty(this.prototype,'title',{
    get: function() {
      // run some magical code
      return this.values.title;
    },
    set: function(value) {
      // run some magical code
      // YIELDING CHANGE EVENTS happens here
      this.value.title = value;
      return value;
    }
  });
}
{% endhighlight %}

Next thing you need to do is to set up some *javascript* which listens to the events being triggerd from "setting" the value of our "title"-property.
Ampersand.js gives us __*ampersand-views*__ to do just that, a little micro-framework which binds the events from model-properties to the DOM.

## What about Object.observe?
In future, the *vanilla javascript* feature "Object.observe" is exactly what does all of this (and more), and it will be supported natively in all major browsers. But... the syntax of Object.observe is quit complex to use in a big project. You'll probably feel the need for some sort of framework to be written arround Object.observe. Ampersand.js does it in an easy way and probably (hopefully) will soon serve as a *polyfill* whenever your browser supports it.


## Ampersand.js vs Backbone
[Backbone](http://backbonejs.org/) is a bulky framework which includes (kind of) everything you will ever need (including [Underscore.js](http://underscorejs.org/) and [JQuery](http://jquery.com/)). Backbone also has it's own syntax which doesn't feel naturally if you only need to update the value of some property 
{% highlight js %}
// Backbone
score.set({title:"My new title"});
// Ampersand.js =)
score.title = "My new title";
{% endhighlight %}
Ampersand.js separates everything in little micro-frameworks which you can require: 
- individually,
- whenever you need them,
- independantly from each other,
- independantly from other frameworks,
- mixed together with other frameworks,
- ...

... so you end up with a build-file which includes ONLY the code you actually use!!!
And best of all: Ampersand.js mimics Object Binding by using getters and setters behind the screens.

> ## TL;DR
> When you start building a new project, a lot of the time you begin to scafold multiple (micro-) frameworks together to end up with a lot of custom scripts to bind one and another together. This often leads to your own bootstrapping-code which more and more becomes to grow and start to look as **your own framework**. 

>If this sounds familiar, have a look at Ampersand.js.
> It's:
- Lightweight
- Performant
- Easy to learn
- Selfexplenatory
- Independant but compatible with any other framework
- Opensource and easily accessible to start your own contribution

> Ampersand.js gives you the standard concepts of other big frameworks like Angular or Backbone (*Models*, *Collections* and *Views*), but it's all conceived as a combination of micro-frameworks. In other words, you only need to require just that little bit of code you actually use, without having to include a bulky, fully fledged, big ass framework, filled with features you'll never use.
> On top of that, the code you write with Ampersand.js looks and feels naturally without having you to learn framework-specific syntax.


## Recommended reading
If you like a quick start for Ampersand.js, you can read the blog of [Ampersand.js in a nutshell](http://silvdb.github.io/2015/04/20/ampersand-js-in-a-nutshell/) by community member (and my colleague/employee), [Silke Van den Broeck](https://twitter.com/Silvdb).
If you like to read more about "Why you should use Ampersand.js", read on with [Introducing Ampersand.js](https://blog.andyet.com/2014/06/25/introducing-ampersand-js) by a member of [&Yet](https://andyet.com/) and co-creator of Ampersand.js [Henrik Joreteg](http://andyet.com/team/henrik)
