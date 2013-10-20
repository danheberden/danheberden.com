---
title: Duck-punching jQuery UI and the Widget Factory
author: dan-heberden
date: 2011-01-15 09:00
template: article.jade
---

jQuery UI is a great user-interface library for jQuery. It has some great
plugins with a fantastic set of options that allow you to perform all kinds of
UI tasks like dialog boxes, tabs, draggable/droppable and more. The CSS
framework of jQuery UI is designed to provide an easy path to change the
appearance of the various widgets and their respective components. 

<span class="more"></span>

Adjusting the styles and acting on events allows for a great deal of
customization. However, some needs simply can't be achieved this way. Instead
of manually modifying a local copy of jQuery UI and serving that custom version
with your web application, a superior route exists using duck-punching and the
widget factory.

Duck-punching is a technique of replacing a function so that it still accepts
and returns the same variables/options, but operates differently. Duck-punching
the functions made available by the widget factory allows the web application
to use a CDN copy of jQuery UI and also persist with newer updates. 

Our example, the jQuery UI Sortable plugin. 

### Demos

Here's how it works now. Re-sort the list using drag and drop. 

<iframe style="width: 100%; height: 190px" height="250" src="http://jsfiddle.net/danheberden/6Kt79/embedded/result,js,html,css"></iframe>

When you let go of the sortable, though, it instantly snaps into place. Lets make it work like this:

<iframe style="width: 100%; height: 190px" height="250" src="http://jsfiddle.net/danheberden/dYyQn/embedded/result,js,html,css"></iframe>

What makes that all possible is taking control over the `_mouseStop` function
inside of the sortable prototype. The widget factory places widgets on
`$.namespace.widgetName` - in this case, `$.ui.sortable`.
On that object's prototype sits those declared functions you see if you browse
the jQuery UI source. 

### The Finished Product

```javascript
// save the old _mouseStop function
var _mouseStop = $.ui.sortable.prototype._mouseStop;

// now lets make a new one
$.ui.sortable.prototype._mouseStop = function(){

  // save copies of the current object and args
  var _this = this;
  var args = arguments;

  // animate the item to its placeholder
  _this.currentItem.animate( this.placeholder.position(), 
    450, "easeOutElastic", function(){
      // call the original function when it's done
      _mouseStop.apply( _this, args );
    });   
}
```

### How it works

```javascript
var _mouseStop = $.ui.sortable.prototype._mouseStop;
```

First, a copy of the current function is saved to the `var _mouseStop`. While we
could copy the original function entirely, if it calls other inner functions it
can get messy and difficult to manage. This way, the new function can perform
whatever operations it wants to and *then* call the original one.

```javascript
$.ui.sortable.prototype._mouseStop = function( event ) 
```

Now that the existing function has been saved to `_mouseStop`, it can be replaced with a new one. 

```javascript
 var _this = this,
       args = arguments;
```

`this` and `arguments` will change inside of animate's callback function (they are function specific) so is necessary to assign them to variables. 

```javascript
_this.currentItem.animate( this.placeholder.position(), 
  450, "easeOutElastic", function() {
```

This is where the magic happens. The `currentItem`, which is the draggable
item, is animated to the location of the placeholder element, that is, the
element creating the blank space between the li's. 

```javascript
_mouseStop.apply( _this, args );
```

Inside of the animation callback function, the original `_mouseStop` function
that was saved in the beginning is called. Using apply, the function can be
sent the same `this` and arguments as the new, modified `_mouseStop` function.

While this might be a contrived example, I hope the principles of overriding
functions on the widget factory came through loud and clear. 

