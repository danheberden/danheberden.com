---
title: "jQuery Plugin: .serializeForm()"
author: dan-heberden
date: 2011-04-01 08:00
template: article.jade
---

More often than not, you want an object of your form data to send to the
server. Some frameworks make heavy use of input names like
`data[ModelName][attribute]`, as well as the common `choices[]` format for multiple form elements.
I needed a way to get that same nested format into
an object I could send to the server in an $.ajax call, so I made one.

<span class="more"></span>

`$.fn.serializeForm` works on any level of nested array syntax in your element
names and creates arrays if items end in `[]`, just like you'd expect.

For more information and source, check out the [github repo](https://github.com/danheberden/jquery-serializeForm).

For a demo, check out: [http://jsfiddle.net/danheberden/3xcTG/](https://github.com/danheberden/jquery-serializeForm)

Example document:

```html
<div id="test">
    <input name="text1" value="txt-one" />
    <input type="checkbox"
                name="top[child][]" value="1" checked="checked" />
    <input type="checkbox"
                name="top[child][]" value="2" checked="checked" />
    <input type="checkbox"
                name="top[child][]" value="3" checked="checked" />

    <select name="another[select]">
        <option value="opt"></option>
    </select>
</div>
```

Running `$( '#test' ).serializeForm()` returns:

```javascript
{
  text1: "txt-one",
  top: {
    child: [ "1", "2", "3" ]
  },
  another: {
    select: "opt"
  }
}
```

\[Update: 2013-03-23. Renamed serializeObject to serializeForm\]
