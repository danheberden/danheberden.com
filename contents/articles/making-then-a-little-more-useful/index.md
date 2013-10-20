---
title: Making Then a Little More Useful
author: dan-heberden
date: 2011-04-02 09:00
template: article.jade
---

Edit 04.06.2011: This feature is now a part of jQuery 1.6 (commit [bb99899c](https://github.com/jquery/jquery/commit/bb99899ca0de93dd12f5a53f409ff6f72bfcf94c)) and
called [.always](http://api.jquery.com/deferred.always/).

The introduction of Deferreds into jQuery in 1.5 is quite a handy feature.
Particularly, being able to make an ajax request and assign callbacks onto the
returned jqXHR. 

<span class="more"></span>

However, at least for me, `.then` doesn't work the way my brain thinks it
should (Thanks to [Ben Alman](http://benalman.com) bringing this up). When you
get a promise back from the deferred, it has three methods: done, fail, and
then

`done` and `fail` do exactly what you'd think: attach functions to be called if
the deferred is resolved or rejected.

`then` is currently a shortcut method for the two, as well as supports the ability
to return another deferred or promise to extend the resolution of the original added
in jQuery 1.8.

```javascript
$.ajax({
    url: 'someUrl'
}).then( doneFn, failFn );
```

The word then, for me, is much more conclusive. It isn't a shortcut, it's an
ultimatum. Thus, I expect `.then` to work like:

```javascript
$.ajax({
    url: 'someUrl'
}).then( willGetCalledNoMatterWhat );
```

So, if you would rather trade in your shortcut for power, be explicit and have
control, or just be part of the cool club - here's how to make .then work the
way you want here's how to add the functionality yourself. I've been persuaded
by Rebecca and Colin to not alter the functionality of an existing, documented
method, but rather, add the additional functionally. Their point was simple: by
changing the functionality of how something existing and documented operates to
work contrary the way it should, anyone stepping into code that had modified
then would have a hell of a time figuring out why it wasn't working to spec.

```javascript
// Wrap in an IIFE and preserve $ incase of $.noConflict();
(function( $ ) {

  // Cache a copy of $.Deferred as it is
  // right now so it can be called later
  var _Deferred = $.Deferred;

  // Overwrite $.Deferred with a new function
  // that takes the same argument
  $.Deferred = function( func ) {

    // Call the original $.Deferred method (the
    // cached copy ) using .apply to assign
    // the same context and send the 'func'
    // argument
    var d = _Deferred.apply( this, func ),
         // copy the created .promise method
         // to duck-punch further down
          _promise = d.promise;

    // add our new 'then' type function, 'always'
    d.always = function( fn ) {
        return d.then( fn, fn );
    };
    // duck-punch the created promise method
    // to use the newly created .always method
    d.promise = function( obj ) {
        var promise = _promise.call( this, obj );
        promise.always = d.always;
        return promise;
    };  

    // return the our modified deferred from
    // _Deferred, but with our new .always
    return d;
  };
})( jQuery );
```

The Result

```javascript
$.ajax({
    url: 'someUrl'
}).always( function() {
   /* i get called no matter WHAT */
});
```
How it works


This duck-punches the existing $.Deferred function with a wrapper function.
`$.Deferred` creates a new deferred object and makes the `.then`, `.done`, and `.fail`
methods on the fly. By saving a copy of the old function, we can let it go
about its usual business and add the `.always` function to both the deferred and
its promise method.


[Demo at jsfiddle.net](http://jsfiddle.net/danheberden/FG8uE/)

Smaller Version

```javascript
(function($){var o=$.Deferred;$.Deferred=function(a){var d=o.apply(this,a),p=d.promise;d.always=function(f){return d.then(f,f);};d.promise=function(j){var n=p.call(this,j);n.always=d.always;return n;};return d;}})(jQuery);
```
