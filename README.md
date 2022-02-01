Welcome!
=============
This is the home for the **Formatted Text** incubation effort. There are several explainers
for this feature:

1. This readme is a general introduction to the problem space and the motivation for this
     effort.
2. Input data model for the API is described in [data model](explainer-datamodel.md).
3. Output data model or [text metrics](explainer-metrics.md).
4. [Rendering](explainer-rendering.md) of the output data model.

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
detailed text metrics for multi-line text (e.g., from previously rendered text in the DOM).
Precise text metrics from the source text can help inform placement of overlays, annotations,
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
formatted. SVG's DOM is slightly more helpful by providing some additional text metrics.

Our goal is to create an abstraction that allows authors to collect multi-line
formatted text into a data model that is independent of the HTML/SVG data model
but can still leverage the power of the browser's native line breaking and text
shaping component. We think it is valuable to re-use formatting principles from CSS
as much as possible. Given the data model, we also want to provide a way to
initiate layout and rendering of that data model to a canvas, and likewise provide
a way to initiate layout and then read-back the text metrics that describe how the text
was formatted and laid out given prescribed layout constraints (without requiring
the text to be rendered at all).

## Related Work

This proposal builds on a variety of existing and proposed specifications in the 
web platform, for whose efforts we are very grateful, and from whom we expect to
get lots of feedback.
* [CSS Text 3](https://drafts.csswg.org/css-text-3/)
* [CSS Inline Layout 3](https://drafts.csswg.org/css-inline-3/)
* [CSS Houdini Layout API](https://drafts.css-houdini.org/css-layout-api-1/)
* [HTML Canvas API and TextMetrics](https://html.spec.whatwg.org/multipage/canvas.html#drawing-text-to-the-bitmap)
   * Open issues: [3994](https://github.com/whatwg/html/issues/3994), [4026](https://github.com/whatwg/html/issues/4026), [4030](https://github.com/whatwg/html/issues/4030), [4033](https://github.com/whatwg/html/issues/4033), [4034](https://github.com/whatwg/html/issues/4034), 
* [SVG Text](https://svgwg.org/svg2-draft/text.html)
* [Proposed Canvas Text Modifiers](https://github.com/fserb/canvas2D/blob/master/spec/text-modifiers.md)
* [CSS Houdini Font Metrics API](https://drafts.css-houdini.org/font-metrics-api/)
* [CSS Houdini Box Tree API](https://drafts.css-houdini.org/box-tree-api-1/)
* [Other platforms for drawing formatted text](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/drawing-formatted-text)
* Google SKIA's Text API Proposals
   * [Text API Overview](https://github.com/google/skia/blob/main/site/docs/dev/design/text_overview.md)
   * [Canvas2D extensions for Shaped Text](https://github.com/google/skia/blob/main/site/docs/dev/design/text_c2d.md)
   * [Shaped Text](https://github.com/google/skia/blob/main/site/docs/dev/design/text_shaper.md)
* ECMA 402 [proposal to expose "line break opportunities"](https://github.com/tc39-transfer/proposal-intl-segmenter-v2) of text which implement the
    [Unicode® Standard Annex #14 UNICODE LINE BREAKING ALGORITHM](https://www.unicode.org/reports/tr14/)

## Implementations

* Chromium has an initial implementation behind the _Experimental Web Platform Features_ flag. See [variations from the explainer in the chromium experimental implementation notes](impl-chromium-notes.md) for additional important details.

## Open issues and questions

Please review and comment on our [existing open issues](https://github.com/WICG/canvas-formatted-text/issues).

## Alternatives Considered

### Fixed Data Model Objects
A previous iteration of this proposal used WebIDL interfaces as containers for a
group of text (`FormattedText` objects), which contained an array of text run objects
(`FormattedTextRun`) with properties for style, etc. 

The prior reason for having an express data model was to enable persistent text
(e.g., similar to DOM's `Text` nodes) to be used for memory-resident updates. However,
experience with DOM `Text` nodes helps us understand that in most cases, the `Text` 
node object itself is irrelevant. Instead, what is needed is the JavaScript string
from the `Text` node in order to mutate, aggregate, and change the text. In the DOM,
such string changes need to be presented via document layout (e.g., "update the 
rendering"), and the only way to do that is to re-add changed strings into DOM `Text`
nodes in an attached document. For Canvas-based scenarios, rendering is not done 
through document layout, but with explicit drawing commands (which then paint when 
rendering is udpated). Therefore having a DOM `Text` node-like data model really did 
not add much value.

Furthermore, simplifying the data model was advantageous for performance-critical 
scenarios that inlcude the creation time of the data model, in addition to layout and
rendering optimizations.

### Intermediate line objects
In a previous iteration of this proposal, we called for a "simple" and "advanced"
model for rendering formatted text to the canvas. The advanced model allowed authors
to place arbitrary lines by alternately measuring a part of the data model and then
rendering the resulting "line metrics" object.

Under that design, we were assuming that authors would want to cache and re-use the
line objects that were produced. If authors would not re-use these objects, then no
optimization could be made for performance (since we were letting authors do the 
fragmentation themselves). For simple use cases where the canvas will be re-painted
without changes, having authors cache and reuse line objects seemed a reasonable 
request. However, if authors decide to only re-paint the canvas when things change, 
then to recapture performance, authors are left to implement their own line invalidation
logic or to just throw away all the lines and start from scratch--the worst-case scenario
for performance. 

We thought about addressing this concern by adding a "dirty" flag to lines that have been
invalidated to help the author create an efficient invalidation scheme. But line analysis 
and caching logic has never been exposed to authors before in the web platform, and we
didn't want to create a feature with this foot-gun. The advanced use case was primarily
about enabling occusions and supporting things like floaters obstructing lines, and CSS
already has standards for those scenarios--when we decided to embrace more CSS constructs
for this feature, it was decided that the advanced use case could be dropped entirely.

**Note:** many of the advanced features possible with this iteration of the proposal 
require new CSS features that are only recently starting to be interoperably implemented
(E.g., CSS Shapes).

### Imperative model
The proposal here addresses two separate problems. One of styling ranges of text
and having an object model and two of auto wrapping text.

An alternative design considered was to support auto wrapping without requiring
that the developer provides all the text upfront. Similar to canvas path, the
developer would call `setTextWrapWidth( availableWidth )` and follow through with
multiple calls to `fillText` on the canvas context that renders text and advances
a cursor forward.

Such an imperative style API was not pursued for two reasons. With bidi text, the
entire width of a right-to-left segment needs to be determined before any of the
`fillText` calls can be rendered. This issue can be addressed by adding a finalization
step say `finalizeFillText()`. However still, such an imperative API adds a performance
cost in terms of recreating the formatted text when the developer is simply trying to
redraw the same text content for a changing available width.

## Privacy Considerations

HTML5 canvas is a browser fingerprinting vector
(see [canvas fingerprinting](https://en.wikipedia.org/wiki/Canvas_fingerprinting)).
Fingerprinting happens through APIs `getImageData`, `toDataURL`, etc. that allow
readback of renderer content exposing machine specific rendering artifacts.
This proposal adds the ability to render multiple lines of text, potential
differences in text wrapping across browsers could contribute to additional
fingerprinting. The existing implementer mitigations in some user agents that
prompt users on canvas readback continues to work here.

We are currently evaluating whether this API would increase fingerprinting surface
area and will update this section with our findings. We welcome any community feedback.

## Contributors:

 [dlibby-](https://github.com/dlibby-),
 [ijprest](https://github.com/ijprest),
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead),
