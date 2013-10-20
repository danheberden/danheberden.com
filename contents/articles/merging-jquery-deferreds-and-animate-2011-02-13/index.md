---
title: "Merging jQuery Deferreds and .animate()"
author: dan-heberden
date: 2011-02-13 10:00
template: article.jade
---

\[2013-10-01: This is still informative, but animate now does this (and more) and `.sub()` is deprecated.\]

jQuery’s animate method, and the shorthand methods that use it, are fantastic
tools to create animations. Creating animations that link together to achieve
a particular effect, and do something specific at the end of the animation, can
be a painful, messy task. Luckily, we have .queue() for mashing animations
together.

<span class="more"></span>

But what happens when you want bridge the gap between ajax requests and
animating? When you want to queue a bunch of animations, get data from the
server, and handle it all at once, without a crap-load of nested callbacks?
That’s when jQuery.Deferred() puts on its cape, tightens its utility belt, and
saves the day.

### Disclaimer

I should note, however, that this is more-or-less giving an example of
a pending feature request to add deferreds support in $.fn.animate. If the
feature request is accepted and landed, it won’t show up until version 1.6 of
jQuery. The principles, however, speak to jQuery’s flexibility and how to forge
its multitude of great features into an even stronger tool.

While this works, its behavior isn’t consistent with that of jQuery. Namely,
the new custom animate method doesn’t return ‘this’, but a Deferred object.
However, that’s kind of the point of $.sub(): allowing you to copy the jQuery
object and have your way with it. So, do try this at home – just don’t threaten
my life if your site explodes.

### The Demo

The following demo is a basic, distilled use-case for this kind of situation.
Clicking the button opens a div that contains a loading message. While it’s
opening, it is also querying the server for information to populate the box.
Once both have finished, the loading div is hidden and the box with the
retrieved data remains.


<iframe src="http://fiddle.jshell.net/danheberden/NMh7c/show/light/" style="height: 190px;"></iframe>


### Modifying .animate()

Here is the code driving the change to .animate()

```javascript
// create a sub of jquery (Basically, a copy we can mess with)
var my$ = $.sub();

// make my$ have a modified animate function
my$.fn.animate = function( props, speed, easing, callback ) {
    // from jQuery.speed, forces arguments into props and options objects
    var options = speed &amp;&amp; typeof speed === &quot;object&quot; ?
             jQuery.extend({}, speed) : {
                complete: callback || !callback &amp;&amp; easing ||
                    jQuery.isFunction( speed ) &amp;&amp; speed,
                duration: speed,
                easing: callback &amp;&amp; easing || easing &amp;&amp;
                    !jQuery.isFunction(easing) &amp;&amp; easing
        };

    // create the deferred
    var dfd = my$.Deferred(),
        // a copy of the complete callback
        complete = options.complete,
        // and the count of how many items
        count = this.length;

    // make a new complete function
    options.complete = function() {
        // that calls the old one if it exists
        complete &amp;&amp; complete.call( this );
        // and decrements count and checks if it's 0
        if ( !--count ) {
            // and when it is, resolves the DFD
            dfd.resolve();
        }
    };

    // all the hooks have been made, call the regular animate
    jQuery.fn.animate.call( this, props, options );

    // return the promise that we'll do something
    return dfd.promise();
};
```
While the comments explain just about everything, be sure to read up on $.sub() if you haven’t already. The new animate function on my$ simulates the method signature of the function, $.fn.animate, it’s attempting to replace. In short, speed, easing, and callback are forced into an options object — the same as if the second parameter was an object.

The deferred is created, a copy of the complete callback, and the count of how
many items. Why? We want to fire the `resolve()` function on the deferred once
all of the animations have finished. The `complete()` callback is replaced with
a wrapper function that calls the original callback and decrements and checks
the count. When the count reaches zero, all items have been animated and it’s
safe to fire the `resolve()` function.

The untouched, standard version of `$.fn.animate` is called with the same `this`
properties, and the modified options object with the new complete wrapper
function.

Returned is the `promise`, `dfd.promise()l`, that lets `$.when()` do its awesomeness.

## Putting it into action

If you haven’t familiarized yourself with deferreds, I highly recommend you
read Eric Hynds’ fantastic article about it.

```javascript
// retrieves content and updates a dom element
// returns a promise
function populateBox() {
    return $.ajax({
        url: 'your/server/url',
        data: { },
        type: &quot;POST&quot;,
        success: function( data ) {
            $('#content').slideDown().html( data );
        }
    });
}

// the &quot;Get Message&quot; click handler
$('button.load').click( function() {

    // save the button, box and loading as my$ objects
    var $button = my$(this).hide(),
        $box = my$('#box'),
        $loading = my$('.loading');

    // when the functions are done
    $.when(
        // $box was created with my$, so it will
        // use the custom animate function
        $box.slideDown(),
        populateBox()
    // then run the 1st function on success
    // and the second function if either fails
    ).then(
        function() {
            // remove loading, we're done
            $loading.slideUp();
        },
        function() {
            // get that button back here
            $button.show();
            // and hide the box
            $box.slideUp();
        }
    );
});
```

Because the new animate function was created on my$, the copy of jQuery made
using `$.sub()`, `.hide()`, `.slideUp()`, and other helper functions that use the
custom .animate() function.

When the animation and ajax request both succeed, thus resolving the promise as
true, the first callback function of `$.then()` is called. However, if one of
them fails, the second will be called. You can re-run the demo (by clicking on
the run button) and opt to fail the ajax request to see the second function get
run. The `.then()` function is a handy mix of `.done()` and `.fail()`, which are
useful if you want to provide multiple callbacks.

### Summary

While I highly doubt this will be the solution to using deferreds with
`.animate()`, I’m confident it’ll get the ball rolling. Too, it covers some other
topics, such as `$.sub()`, deferreds, and wrapping functions with alternate
behavior, that hopefully you found interesting.
