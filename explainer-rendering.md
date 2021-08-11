# Formatted Text - Rendering

An approach to rendering multi-line formatted text to an HTML 2D canvas context.

This explainer focuses on rendering [the data model](explainer-datamodel.md) for formatted text.
For a general overview of the problem space, see the repo's [readme](readme.md). In addition to rendering,
the data model can [provide metrics](explainer-metrics.md).

## `drawFormattedText`

To render the `FormattedText` data model, it must first be formatted with the `format` API (see the 
[text metrics explainer](explainer-metrics.md)). Authors will need to leverage the resulting basic 
metrics to compute the placement (`x` and `y` on the canvas) based on the final formatted text's 
total width and height. This makes sense since "height" (which is `blockSize` in horizontal writing
modes and `inlineSize` in vertical writing modes) is unknown before the text has been formatted.

Two options are available for drawing the post-formatted `FormattedText` metrics objects to a 2D canvas.
Drawing these same objects in a WebGL or WebGPU canvas/texture is TBD, but we understand that the 
provided glyph `id` (into the font itself) is needed as well as `advance` data, which are both provided
for each glyph.

| API extensions to [CanvasText](https://html.spec.whatwg.org/multipage/canvas.html#canvastext) | Description |
|---|---|
| .`drawFormattedText`(`formattedTextParagraph`,`x`,`y`) | draws the formatted paragraph onto the canvas, where the `x` and `y` are the upper-left corner in which to place the resultant paragraph. |
| .`drawFormattedText`(`formattedTextLine`,`x`,`y`) | An overload that performs the same function, but when given a line object instead. |

An example usage of `drawFormattedText`:

```js
// Setup the data model
let formattedText = new FormattedText("the quick ", "brown", " fox jumps over the lazy dog");
formattedText.textruns[1].styleMap.set('font-weight', 'bold');

// format/layout the data model given an inline width constraint
let paragraph = await formattedText.format( /*wrapWidth*/ 250 );

// Render to a canvas at (0, 50)
context.font = "18pt Arial";
context.drawFormattedText( paragraph, /*x*/0, /*y*/50 );
```

This would produce the following output on the canvas:

![Wrapped text rendered in a canvas.](explainerresources/Example1.png)

### Bidi Text

No additional work is needed from web developers to support bidi text.
Implementations will perform bidi analysis on the `FormattedText`'s text runs
and create internal bidi runs if necessary. An example demonstrating bidi text follows:

```js
context.font = "30px Arial";
context.drawFormattedText( await new FormattedText( "Sample arabic بمدينة مَايِنْتْس، ألمانيا text." ).format(350) , 0, 30 );
```

produces the following output on the canvas:

!["Wrapped text rendered in a canvas."](explainerresources/Example2.png)

# QnA

## How are the layout constraints specified?

In the proposed text metrics, the layout constraints are applied via a call to `format` on the 
data model. Thus, this rendering explainer need only focus on how to render the "already processed"
text.

## What are the initial values for CSS properties, esp. when used with the Canvas. Who wins?

Initial values for CSS properties will be as-specified in CSS (we don't intend to mess with that.

⚠🚧 Our current thinking is that the rendering of formatted text will completely ignore the current 
Canvas drawing state, perhaps with one exception--the default font. To this end, our rendering
APIs will not follow the conventional `fill...` and `stroke...` patterns, to help make it more 
clear that they are unique from the rest of the Canvas state. We welcome author feedback on this
front and use cases where it might be useful to respect certain canvas state (i.e., current 
transform).

## CSS unit value resolution (percentages, px, vh, etc.)

These units are evaluated against the inline direction constraints at `format` time. For block
direction, an infinitely-deep block is assumed.

⚠🚧 Are there use cases for using `vh` units for paragraphs of text?

## What about a glyph drawing API?

⚠🚧 We are seeking to understand use cases for per-glyph drawing and what metrics information
would be required to support it.

## Accessibility Considerations for Rendering

As a potential render target for `FormattedText`, the canvas itself is a persistant challenge for 
accessibility on the web today. Several efforts are underway, including a
[promising solution](https://github.com/WICG/aom/blob/gh-pages/explainer.md#use-case-4-adding-non-dom-nodes-virtual-nodes-to-the-accessibility-tree)
as part of the Accessible Object Model (AOM) family of proposals.

Meanwhile, web developers are encouraged to use the canvas element's fallback content
to provide HTML markup that describes the canvas' current state until or unless a better
solution that utilizes the `FormattedText` directly is established. For now, the aggregate 
text of a `FormattedText` object's text runs should be placed in block-styled
[flow content](https://html.spec.whatwg.org/multipage/dom.html#flow-content-2) (such
as a `<p>` element), with formatted sections wrapped in appropriate
[phrasing content](https://html.spec.whatwg.org/multipage/dom.html#phrasing-content-2)
(such as `<span>` and styled to match the `FormattedTextRun` formatting.) The
markup should make use of
[ARIA Live Regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)
to be sure assistive technologies (ATs) detect and announce any dynamic changes.

## Metrics for FormattedText and layout calculations
The [metrics explainer](explainer-metrics.md) describes how to take the data model 
representation of formatted text and extract metrics for relating the laid-out text 
back to the formatted text data model.

## Contributors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
