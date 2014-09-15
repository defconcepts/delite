---
layout: default
title: Delite and Custom Elements
---

# Delite and Custom Elements

The heart of delite is its support for custom elements.
In a nutshell, this means that an app has the ability to define new tags (often called widgets),
for example `<my-combobox>`, which can be used interchangeably with the native HTML tags like `<input>`.

Delite's support of custom elements is mainly split across four components:

* [register](register.html) - utility for registering and instantiating new widgets
* [Widget](Widget.html) - base class for all widgets
* [handlebars!](handlebars.html) - AMD plugin to compile templates
* [theme!](theme.html) - AMD plugin to load CSS for this widget for the current theme


## Defining Custom Elements

Custom Elements are defined using [register()](register.html).
The [register()](register.html) method is a combination of a class declaration system
(internally it uses [dcl](http://dcljs.org)),
and a shim of [document.registerElement()](http://www.w3.org/TR/custom-elements/)
from the new custom elements proposed standard.

When defining a custom element you generally:

* Define which HTML class you are extending.  Usually this is `HTMLElement`.
* Define a template.
* Define CSS for your custom element, preferably one CSS file for each supported theme.
* Define the public properties of your custom elements.

There are other optional steps, like defining methods and emitting custom events, but those are the most basic.

Example:

{% raw %}
```js
define([
	"delite/register",
	"delite/Widget",
	"delite/handlebars!./MyWidget/MyWidget.html",				// the template
	"delite/theme!./MyWidget/themes/{{theme}}/MyWidget.css"		// the CSS
], function (register, Widget, template) {
	register(
		"my-widget",				// the custom tag name
		[HTMLElement, Widget],		// the superclasses
		{
			// my template
			template: template

			// my public properties
			stringProp: "",
			numProp: 123,
			boolProp: true,
			objProp: null
		}
	);
});
```
{% endraw %}

The template (MyWidget.html) will be something like:

{% raw %}
```html
<template>
	Hello {{stringProp}}!  You have {{numProp}} messages and
	{{this.boolProp ? "no" : "some"}} calendar events.
</template>
```
{% endraw %}

It is discussed in detail in [handlebars!](handlebars.html).


## Instantiating Custom Elements Declaratively

Like native elements, custom elements can be instantiated declaratively.
Given a custom element definition like above, you would declare an instance like:

```html
<my-widget stringProp="hi" numProp="456" boolProp="false"
	objProp="myGlobalVariable"></my-widget>
```

Note that delite does automatic type conversion from the attribute value (which is always a string)
to the property's type.

### Parsing

In order for declarative custom elements to be instantiated on platforms without native custom element support,
you must call the parser:

```js
require(["delite/register", "requirejs-domReady/domReady!"], function (register) {
	register.parse();
});
```

Note that on platforms *with* custom element support, the custom elements will be instantiated before
the call to `register.parse()`, and without any guaranteed order.  Therefore, if your custom elements
depend on a global variable, like in the example above, you should make sure it is available before
the custom element is loaded.   Therefore, you may need code like this:

```js
require(["dstore/Memory"], function (Memory) {
	myGlobalVar = new Memory();
	require(["delite/register", "requirejs-domReady/domReady!"], function (register) {
		register.parse();
	});
});
```
### Declarative Events

Like native elements, custom elements' main notification mechanism is emitting events.
Declarative markup has a special syntax for listening to events, both natural events
and synthetic ones:

```html
<my-widget on-click="console.log(event);"
	on-element-selected="console.log(event);"></my-widget>
```

The example above is listening for a click event as well as a synthetic "element-selected" event.

Note how there is a convenience `event` parameter available to be referenced in the inlined code.

Of course, you can also use `Element.addEventListener()`, but this syntax is often convenient.

## Creating Custom Elements Programatically

Custom elements can be created programatically the way that plain javascript objects are instantiated:

```js
var myWidgetInstance = new MyWidget({
	stringProp: "hi",
	numProp: 456,
	boolProp: false,
	objProp: myGlobalVariable
});
```

Custom elements can also be created programatically just like native elements, except that
[to support browsers without native custom element support] you
need to call the `register.createElement()` shim rather than calling `document.createElement()`:

```js
var myWidgetInstance = register.createElement("my-widget");
myWidgetInstance.stringProp = "hi";
myWidgetInstance.numProp = 456;
myWidgetInstance.boolProp = false;
myWidgetInstance.objProp = myGlobalVariable;
```

The first syntax calling `new MyWidget({...})` is just syntactic sugar for the second syntax.

Note that:

* We are setting properties, not attributes.
* We are using the correct types, not setting the properties to string values.

During creation you also typically set up event handlers, like:


```js
myWidgetInstance.on("click", myCallback1);
myWidgetInstance.on("selection-change", myCallback2);
```

Finally, note that once the element is inserted into the DOM, and any other related DOM nodes (typically child
nodes) have been inserted, you need to call `startup()`:

```js
myWidget.placeAt(document.body);
myWidget.startup();
```