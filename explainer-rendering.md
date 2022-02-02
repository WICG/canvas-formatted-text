# Formatted Text - Rendering

This explainer describes how to render formatted text. See also: 
[specifying the data model](explainer-datamodel.md) for formatting, and the 
[output results/ metrics](explainer-metrics.md).

There may be multiple rendering targets. This document describes rendering formatted text to 
a 2D canvas.

## Prerequisites

Rendering formatted text first requires raw text and formats to be laid out via the `format` API
(see the [data model explainer](explainer-datamodel.md)). The results of `format` are then potential
objects for rendering. As noted in the [output results/ metrics explainer](explainer-metrics.md),
`format` results consist of `FormattedText`, `FormattedTextLine`, and `FormattedTextFragment` objects.
Each of these objects can be rendered independently through the APIs described below.

## `drawFormattedText`

A new API is added to the [CanvasText](https://html.spec.whatwg.org/multipage/canvas.html#canvastext) mixin.

```js
 // Draw formatted text accepts output from the format function
 context.drawFormattedText( FormattedText.format( "hello world!" ), 50, 50 );
```

The second and third parameters are the `x` and `y` coordinates on the canvas to draw the formatted content.

An example usage of `drawFormattedText`:

```js
// Setup the data model
let formattedText = FormattedText.format( [ "the quick ", 
                                            { text: "brown", style: "font-weight:bold" },
                                            "fox jumps over the lazy dog"
                                          ], "font:18pt Arial", { width: 250 } );
// Render to a canvas at (0, 50)
context.drawFormattedText( formattedText, /*x*/0, /*y*/50 );
```

This would produce the following output on the canvas:

![Wrapped text rendered in a canvas.](explainerresources/Example1.png)

# QnA

## What are the initial values for CSS properties, esp. when used with the Canvas. Who wins?

Initial values for CSS properties will be as-specified in CSS.

⚠🚧 Our current thinking is that the rendering of formatted text will completely ignore the current 
Canvas drawing state. Drawing formatted text is more similar to drawing a rectangular image than to 
stroking or filling text using `strokeText` and `fillText`. We welcome author feedback on this
front and use cases where it might be useful to respect certain canvas state (i.e., current 
transform).

⚠🚧 Are there use cases for using `vh` units for paragraphs of text?

## What about a glyph drawing API?

⚠🚧 We are seeking to understand use cases for per-glyph drawing and what metrics information
would be required to support it.

## Accessibility Considerations for Rendering

Canvas is a persistant challenge for accessibility on the web today. Several efforts are underway,
including a
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

## Contributors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
