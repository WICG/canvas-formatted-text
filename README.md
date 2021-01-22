Formatted Text
=============
An imperative approach to multi-line text; its data model, rendering, and metrics.

## Introduction & Challenges

### Imperative multi-line text lacking in canvas

Applications like word processors, spreadsheets, PDF viewers, etc., face a
choice when moving their legacy presentation algorithms to the web. The view layer of
these applications can be designed to output HTML, SVG or use Canvas. Canvas
is an expedient choice for some view models since it can easily present data models
where the view model and the data model are not tightly coupled. Additionally, the
Canvas APIs map well to the imperative APIs the legacy algorithms used for rendering
content.

In gaming and mixed reality (VR and AR) scenarios today, Canvas is the only logical
choice for presenting a rapidly-changing view.

In these scenarios, it is often necessary to present text to the user. The 2D Canvas
API currently provides a relatively simplistic text rendering capability: A single run
of text can be
[measured](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/measureText)
and [rendered](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/fillText)
**as a single line**, optionally compressed to fit
within a certain width. If the developer needs to present more than just a few words
(e.g., a paragraph of text) then things get very complicated very quickly. Some options
might include trying a hybrid approach of overlaying HTML elements with the Canvas to
leverage HTML's native text rendering and wrapping capabilities, or attempting to
write your own line breaking logic in JavaScript using the primitive Canvas text measuring
and rendering APIs as a starting point. Neither of these options are very desirable,
and the HTML hybrid approach may not be available depending on the scenario (e.g., in
VR headsets).

### Lack of unified metrics for multi-line text in HTML

In other scenarios, separately from Canvas, applications are interested in getting more
detailed metrics for multi-line text (e.g., from previously rendered text in the DOM).
Precise metrics from the source text can help inform placement of overlays, annotations,
translations, etc. Unfortunately, text metrics that help map source text placement in a
given layout are not currently provided by the web platform. As a result a variety of 
workarounds are employed to try to approximate this data today often at the cost of 
slower performance for users.

It is our aspiration to bring a unified handling of multi-line text metrics to the web
platform that will service the imperative data model for canvas scenarios as well as 
address many other use cases for text metrics in HTML.

### Challenges for JavaScript implementations

Why would a JavaScript implementation of line-breaking and wrapping be hard? While this
may seem trivial at first glance, there are a number of challenges especially for
applications that must be designed to handle rendering of text in multiple languages.
Consider the complexity involved in the following requirements for a robust line
breaking algorithm:

* Identify break opportunities between words or
  [Graphemes](https://en.wikipedia.org/wiki/Grapheme). Break opportunities are
  based primarily on the Unicode Spec but also use dictionaries for languages
  like Thai and French that dictate additional line breaking rules.
* Identify grapheme clusters. Graphemes are character combinations (such as
  [Diacritics](https://en.wikipedia.org/wiki/Diacritic) and
  [Ligatures](https://en.wikipedia.org/wiki/Orthographic_ligature)) that result
  in a single glyph and hence should not be broken up.
  E.g.: `g` (Latin small letter G 0067) + `◌̈ ` (Combining dieresis 0308) = `g̈`
* Handle Bidi text. For proper Bidi rendering the bidi level context needs to be
  considered across lines.
* Text Shaping and Kerning. These features can affect the measured pixel length
  of a line.

Javascript libraries could perform line breaking, but as noted above, this is
an arduous task. This gets more complicated if text with different formatting
characteristics (e.g., size, bold, italic) are needed. 

## Goals

The browser already has a powerful line breaking, text shaping component used
for regular HTML and SVG layout. Unfortunately, this component has been tightly coupled
into the browser's rendering pipeline, and not exposed in an imperative way. 
Furthermore HTML's DOM provides only a very limited set of metrics for text elements
and almost no insight into the post-layout details of how the source text was
formatted. SVG's DOM is slightly more helpful by providing some additional metrics.

Our goal is to create an abstraction that allows authors to collect multi-line
formatted text into a data model that is independent of the HTML/SVG data model
but can still leverage the power of the browser's native line breaking and text
shaping component. We think it is valuable to re-use formatting principles from CSS
as much as possible. Given the data model, we also want to provide a way to
initiate layout and rendering of that data model to a canvas, and likewise provide
a way to initiate layout and then read-back the metrics that describe how the text
was formatted and laid out given prescribed layout constraints (without requiring
the text to be rendered at all).

## Related Work

This proposal builds on a variety of existing and proposed specifications in the 
web platform, for whose efforts we are very grateful, and from whom we expect to
get lots of feedback.
* [CSS Inline Layout 3](https://drafts.csswg.org/css-inline-3/)
* [CSS Houdini Layout API](https://drafts.css-houdini.org/css-layout-api-1/)
* [HTML Canvas API and TextMetrics](https://html.spec.whatwg.org/multipage/canvas.html#drawing-text-to-the-bitmap)
* [SVG Text](https://svgwg.org/svg2-draft/text.html)
* [Proposed Canvas Text Modifiers](https://github.com/fserb/canvas2D/blob/master/spec/text-modifiers.md)
* [CSS Houdini Font Metrics API](https://drafts.css-houdini.org/font-metrics-api/)
* [CSS Houdini Box Tree API](https://drafts.css-houdini.org/box-tree-api-1/)
* [Other platforms for drawing formatted text](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/drawing-formatted-text)

# Explainers

We will be pursing consensus on each of the three aspects of the formatted text 
problem space, starting with the data model and then detailing how it can be rendered
and how it's metrics can be extracted:
* [Formatted Text Data Model](explainer-datamodel.md)
* [Formatted Text Rendering](explainer-rendering.md)
* [Formatted Text Metrics](explainer-metrics.md)

# Contributors:

 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
