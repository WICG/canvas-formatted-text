Formatted Text - Data Model
=============
An object model for describing multi-line formatted text that does not use DOM Text 
nodes or Elements (does not require DOM).

This explainer focuses on the data model for formatted text. For a general overview 
of the problem space, see the repo's [readme](README.md). Given this data model, there are 
additional explainers for [how to render it](explainer-rendering.md) and 
[how to obtain metrics from it](explainer-metrics.md).

## Providing an input model for formatted text

On the web today, if you want to render text, you essentially have two options (generally): use
the DOM (HTML's CharacterData/Text nodes or SVG Text-related Elements), in which case you put the
text into the "retained" mode infrastructure of the web platform and it decides when and how
to compose and render the text with declarative input from you (in the form of CSS); or you can
use Canvas and "write" the text when and how you want with JavaScript (an "immediate" mode 
approach). The Canvas provides very limited text support today and (by design) leaves any special
formatting, text wrapping, international support, etc., up to the JavaScript author.

We propose a hybrid approach that allows the author to retain the benefits of the DOM's "retained"
mode objects, reusing the web platform's existing line formatting and wrapping engine along with 
support for CSS, while also enabling authors to apply the objects to a Canvas how and when they
choose. Alternately, authors may not desire to render the objects to a Canvas at all, and instead
can run a given layout over them to obtain metrics useful in other scenarios.

In this explainer, a new formatted text data model is presented (the "retained" mode objects) which
is the minimal necessary objects to allow CSS-styled text runs to be collected in a sequence
for layout processing.

### Principles
* An imperative JavaScript-friendly text representation.
* Scope to the needs of inline text layout.
* Leverage CSS as the universal text layout system for the web (even in an imperative model) which also provides for future extensibility.
* Avoid multi-level hierarchal text structures (which must be linearly iterated for rendering anyway)
* Object-centric model (versus range+indexes) to improve encapsulation and avoid problems with overlapping formatting.

### Primary Objects

The data model for formatted text is quite simple and involves only two objects: `FormattedText`
is a container that orders a list of `FormattedTextRun` objects that contain the actual text and
formatting information. (The `FormattedText` container can also have formatting applied that may
inherit down to the `FormattedTextRun`s.)

To maximize authoring convenience, we are proposing using the 
[ObserverableArray pattern](https://heycam.github.io/webidl/#idl-observable-array) to allow authors
to manage the contained array exactly like a JavaScript Array.

<img src="explainerresources/formatted-text-om.png" alt="FormattedText object holds an array of FormattedTextRun objects through a property called 'textruns'." align="center"/>

## Creating FormattedText and FormattedTextRun objects

`FormattedText` and `FormattedTextRun` objects are created using the 'new' pattern:

```js
let text = new FormattedText();
let textrun = new FormattedTextRun();
```

Attach `FormattedTextRun` objects to their `FormattedText` object's `textruns` property (an Array):

```js
text.textruns.push( textrun );
```

Add text to the `FormattedTextRun` object:

```js
textrun.text = "hello";
```

To simplify creation of a `FormattedText` object and its textruns, the constructors also supports
various overloads:

```js
let text = new FormattedText("hello", " world");
// equivalent to:
let text = new FormattedText();
text.textruns.push( new FormattedTextRun( {text: "hello"} ) );
text.textruns.push( new FormattedTextRun( {text: " world"} ) );
// make a copy of all the text runs from another FormattedText object to be included in a new one:
let text2 = new FormattedText( text.textruns );
// copy everything from another FormattedText object and its text runs
let text3 = new FormattedText( text2 );
```

## Common Styles

`FormattedTextRun` objects have two styling shortcuts that can be set at contruction time:
* **font** - Any string accepted by the CSS text of the *font* property
* **color** - Any string accepted by the CSS text of the *color* property

```js
let textrun2 = new FormattedTextRun( { text: "hello, I'm green with envy", font: "20pt Courier", color: "#00FF00" } );
```
These values can also be read and changed after the object is created:

```js
textrun2.font = "10pt Courier";
textrun2.color = "#00CC00";
```

## CSS Styling of FormattedText and text runs
A wide range of inline layout-related CSS is supported on both the `FormattedText` and
`FormattedTextRun` objects.

To set these use the `styleMap` property (just like in HTML):

```js
textrun2.styleMap.set("text-decoration", "underline");
```

The `styleMap` property is available on both the `FormattedText` and `FormattedTextRun` objects.

It is also possible to set multiple styles at once similar to how HTML parses the `style` attribute:

```js
textrun2.setStyle( "color: yellow; font: 15pt Verdana" );
```

### Cascading of values from FormattedText to text runs
CSS properties applied to the style map from the FormattedText that are specified to inherit
from parent to child will do so from the FormattedText to it's text run child objects.

## Simple Example

In this example, we create a `FormattedText` object and its text runs that we eventually 
want to render with the text "The quick **brown** fox jumps over the lazy dog" where the
word "brown" is colored brown and bold. In the [rendering explainer](explainer-rendering.md) we
add layout constraints and render it to a canvas; at present, while it resides in the data model
it has no associated layout.

```js
// Collect text into a FormattedText object
let formattedText = new FormattedText( "The quick ", "brown", " fox jumps over the lazy dog" );
formattedText.textruns[1].setStyle( "color: brown; font-weight: bold" );
```

The above is the semantic equivlant to the following declarative HTML markup:

```html
<div>
  The quick <span style="color: brown; font-weight: bold">brown</span> fox jumps over the lazy dog
</div>
```

Where the `<div>` element is the container for an inline formatting context (and can be styled), 
and the `<span>` contains the formatting for the word "brown". More precisely, since each 
`FormattedTextRun` object can be styled, the following markup better represents the semantic 
equivalent:

```html
<div>
  <span>The quick </span>
  <span style="color: brown; font-weight: bold">brown</span>
  <span> fox jumps over the lazy dog</span>
</div>
```

## CSS to achieve advanced scenarios

### Vertical Text

Show example

### Occlusions/ Non-rectangular wrapping

Show example

### Special Formatting

Show example

## WebIDL (for nerds and implementers)

```webidl
[Exposed=Window,Worker] 
interface FormattedText { 
  constructor();
  constructor(DOMString textRunText...);
  constructor(sequence<FormattedTextRunInit> textRunInit);
  constructor(FormattedText copyConstructorInit);

  attribute ObservableArray<FormattedTextRun> textruns; 
}; 
FormattedText includes FormattedTextStylable; 

dictionary FormattedTextRunInit { 
  DOMString text = ""; 
  DOMString font = "";
  DOMString color = "";
}; 

[Exposed=Window,Worker] 
interface FormattedTextRun { 
  constructor();
  constructor(DOMString text);
  constructor(FormattedTextRunInit textRunInit);

  attribute DOMString text; 
  attribute DOMString font; 
  attribute DOMString color; 

  // owning FormattedText (if any) 
  readonly attribute FormattedText? owner;         // null if not contained in a FormattedText
 
  // navigation in the array 
  readonly attribute FormattedTextRun? next;       // null if last or not contained in FormattedText
  readonly attribute FormattedTextRun? previous;   // null if first or not contained in FormattedText
}; 
FormattedTextRun includes FormattedTextStylable; 
 
interface mixin FormattedTextStylable { 
  [SameObject] readonly attribute StylePropertyMap styleMap;
  void setStyle(USVString cssText);
}; 
```

## Supported CSS on FormattedText and text runs

What's possible and what's not, for both the FormattedText and for its text runs.

| CSS Property | FormattedText | FormattedTextRun |
|--------------|---------------|------------------|
| text-decoration |            | ✔ |

## Rendering the FormattedText
The [next explainer](explainer-rendering.md) describes how to take the data model representation of
text and make is show up on screen.

## Authors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
