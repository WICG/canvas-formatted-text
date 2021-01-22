Formatted Text - Data Model
=============
An object model for describing multi-line formatted text that does not use DOM Text 
nodes or SVG elements (does not require DOM).

This explainer focuses on the data model for formatted text. For a general overview 
of the problem space, see the repo's [README.md]. Given this data model, there are 
additional explainers for [how to render it](explainer-rendering.md) and 
[how to compute metrics for it](explainer-metrics.md).

## Providing an input model for formatted text

### Principles
* An imperative JavaScript-friendly text representation.
* Scope to the needs of inline text layout.
* Leverage CSS as the universal text layout system for the web (even in an imperative model) which also provides for future extensibility.
* Avoid multi-level hierarchal text structures (which must be linearly iterated for rendering anyway)
* Object-centric model (versus range+indexes) to improve encapsulation and avoid problems with overlapping formatting.

### FormattedText and text runs

The data model for formatted text is quite simple and involves only two objects: `FormattedText`
is a container that orders a list of `FormattedTextRun` objects that contain the actual text and
formatting information. (The `FormattedText` container can also have formatting applied that may
inherit down to the `FormattedTextRun`s.

To maximize authoring convenience, we are proposing using the ObserverableArray pattern in order
to allow authors to manage the contained array exactly like a JavaScript Array.

(image of the data model here)

## Creating FormattedText and FormattedTextRun objects



## CSS Styling of FormattedText and text runs
What's possible and what's not, for both the FormattedText and for its text runs.

A FormattedText object represents a block container in CSS that establishes 
an *inline formatting context* for it's content.


### Cascading of values from FormattedText to text runs
CSS properties applied to the style map from the FormattedText that are specified to inherit
from parent to child will do so from the FormattedText to it's text run child objects.

## Simple Example

Consider a use case where a web developer wants to eventually render the text “the quick
**brown** fox jumps over the lazy dog” into a horizontal space 250px wide and have
the text wrap instead of compress-to-fit. In order to enable
this scenario, we introduce a new object `CanvasFormattedText` which will keep track
of all the formatted text. We also introduce two new formatted text drawing APIs onto
the canvas context: `strokeFormattedText` and `fillFormattedText`.

First we will consider how to represent the bolded text in the example above?
Regular HTML uses markup elements to style parts of text. For the 2d Canvas API
we split the text into "runs" and allow each run to have its own optional formatting:

```js
const c = document.getElementById( "myCanvas" );
const context = c.getContext( "2d" );

// Collect text into a CanvasFormattedText object
let formattedText = new CanvasFormattedText();
formattedText.appendRun( { text: "the quick " } );
formattedText.appendRun( { text: "brown", font: `bold ${ context.font }` } );
formattedText.appendRun( { text: " fox jumps over the lazy dog" } );
```

## CSS to achieve advanced scenarios

### Vertical Text

Show example

### Occlusions/ Non-rectangular wrapping

Show example

### Special Formatting

Show example

## Rendering the FormattedText
The [next explainer](explainer-rendering.md) describes how to take the data model representation of
text and make is show up on screen.





## Authors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
