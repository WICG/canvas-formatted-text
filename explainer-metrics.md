# Formatted Text - Metrics

A representation of post-layout metrics for inline layout content, in particular the
metrics from the formatted text data model.

This explainer focuses on the metrics that are offered for inline text content, in particular
the data model for formatted text. We aspire to create these metrics in a way that allows them
to be supported for other sources of inline text content, in particular the DOM or a Worklet's
Layout API algorithm.
Additional scenarios include different web application rendering systems which can make use of
the text shaping information to perform their own rendering logic. For example WebGL based apps 
with text content. The metrics would be used to determine how to correctly position glyphs in a
typographically correct manner.

We took extensive inspiration from the [Text API explainer](https://github.com/google/skia/blob/main/site/docs/dev/design/text_overview.md),
and the use cases supporting the rendering of glyphs as described therein, as well as the
notion of a Position object.

For a general overview of the feature, see the repo's [readme](README.md).
You can also learn more about the [formatted text data model](explainer-datamodel.md) and 
[how to render it](explainer-rendering.md).

# Use cases

## Peparing text for rendering: from gross placement needs to fine-grained glyph control

### Use case: paragraph placement

This use case is the most basic use case we can imagine--identifying the placement of some
Formatted Text into a view layer (like Canvas). Placement needs two things, a reference 
coordinate (x/y) and size metrics (bounding box of width/height).

Author provides: 

* input `FormattedText` object (including styled runs, where styles may include runs with
   differing font characteristics, line-spacing, justification rules, etc.) 
* line-wrap constraints (width constraint in horizontal languages) 

Metrics provide: 

* Final shaped and formatted paragraph width and height. 

A rendering API must provide: 

* (x, y) location to place the formatted paragraph. (Authors ensure paragraph will fit in 
   the space provided by their data model. If not, they can adjust font-size, line-width, 
   line-spacing, etc., on the original `FormattedText` objects and re-request metrics until
   the desired goal is met.

### Use Case: line placement

In this case, the author would like the platform to calculate line wrapping, but intends to
render each line iteratively (such as for captions), or with custom spacing, etc.

Author provides same information as above. 

Metrics provide: 
* Access to formatted line objects with width and height (including their offsets from the
   paragraph container)
* Pointers back to the input characters for the bounds positions of each line.

A rendering API must provide: 

* (x, y) location to place formatted line 

### Use Case: specific glyph placement

⚠🚧 We would like to validate this use case for Canvas 2D scenarios. For WebGL scenarios, we
understand the key information needed for rendering is the given shaped font's glyph id and
glyph advance information. Is a Canvas 2d rendering API needed? A sketch of how this might 
work follows.🚧⚠

Author provides same information as in the previous use case. 

Metrics provide: 

* List of Shaped Glyph metrics per fragment (fragment is a unit of glyphs that all share the
   same format/font/bidi/etc.). 
* Pointers back to the input characters for each glyph's bounds. 

A rendering API must provide:

* (x, y) location to place a glyph, given a Fragment (holder of glyph's shaped information),
   and glyph info (index within that fragment or ID within the font)

## Editing scenarios for inline text

Many of the scenarios behind the chosen metrics are based on common editing use cases. An editing
surface must provide a visual view and the means to move insertion points, selection ranges, etc.,
by responding to various input including pointing devices and keyboard. In order to support these
input modalities, the metrics supplied by the explainer chiefly provide the means of understanding
the relationships between parts of text as it was laid out (the glyphs that make up segments of
like-formatted runs called "fragments" in this proposal, lines, etc.) and the relatiionship between
these metrics and their source `FormattedText` objects.

### Rendering a selection over text (and placing/moving a caret)

Author provides: 

* Input data model (same as previously described). 
* JS objects for tracking selection anchor and focus locations in the input data model (i.e., 
   reference to a `FormattedTextRun` and character offset).

Metrics provide:

* Position objects that map line/fragment/glyph indexes to input data model runs/offsets.
   (Map from text metrics to data model.)
* API for obtaining position objects given input data model runs/offsets. (Map from 
   data model to text metrics.)
* API for obtaining position objects given (x,y) offsets relative to the formatted paragraph.
   (Map for mouse/touch/pen input to text metrics and data model.)
* Access to formatted line objects with width/height (bounding box) and offsets from their
   container. 
* Access to formatted fragments within lines with width/height (bounding box) and offset from
   their container (if a selection needs to be tightly bound around the formatted glyph runs
   inside of lines).
* Access to glyph width/height (bounding box) and offset from the fragment container. 

A rendering API does not need to provide specific features (other than those noted in previous 
use cases) for this scenario. (e.g., rendering a selction and caret can be done with existing
APIs).

# Overview: data model to metrics to rendering

The [data model](explainer-datamodel.md) itself cannot be rendered to a canvas as-is. We
experimented with the idea of directly-rendering a `FormattedText` object, and found that in
nearly every scenario the author needed to know both the expected width (i.e., inline-size) and
the resultant height (i.e., block-size) in order to place the formatted content properly in the
canvas.

As is the case with HTML text in normal flow, vertical positioning options for paragraphs of text
are limited. CSS Flex and Grid now offer the desired alignment properties, however, it is not our
goal to introduce these new CSS layout models to `FormattedText`. Instead, we will let authors
calculate the placement of the formatted text themselves. To do this, the metrics API will provide
both the inline and **block size** values.

In order to get the data model's inline and block size, the `FormattedText` object must be… well, 
formatted.

A new API is added to the `FormattedText` object: **format()**. This API takes a maximum inline size 
parameter and asynchronously returns a **formatted paragraph object** containing the inline size and
block size (among other things) after running all shaping, line breaking, and formatting of the text.
This paragraph object **is a snapshot** of metrics for the `FormattedText` data model, given the
constraints applied at the time of formatting, and is **not updated** as additional changes are made
to the data model.

The formatted paragraph is a container for all the input data model's metrics. It contains the APIs 
to get additional line, fragment, and glyph information. The object hierarchy is shown below (note
the image shows lines in a horizontal writing mode--but vertical writing modes are supported):

![A FormattedTextParagraph box contains four horizontal FormattedTextLine objects. Each line object contains one or more FormattedTextFragment objects. Each fragment object is a container for glyph information.](explainerresources/metrics-structure.png)

These objects (that contain a snapshot of metrics and layout information) may be rendered independently.
We suggest APIs to render the entire paragraph, a single line, or (needs validating) any sequence of
glyphs from a fragment (see [Rendering section](explainer-rendering.md)).

| New APIs on `FormattedText` | Description |
|---|---|
| .`format`(`inlineSize`) | Asynchronously formats the `FormattedText` object, returning an object suitable for rendering and extracting metrics: a `FormattedTextParagraph` |

## Metrics lifetime expectations

⚠🚧 We encourage prototyping to get feedback about the implementation opportunities or complexities 
of this suggested approach.

It seems likely that developers will want to `format` their `FormattedText` frequently (for example, 
as the model is changed to respond to user actions). Because metrics objects are snapshots, this 
could lead to an accumulation of many copies of metrics, only the most recent of such is relevant to
the latest data model udpates at any given time. One approach, to avoid unnecessary pressure on the 
garbage collector, is to return the same instance of one or all of the metrics objects each time
`format` is called. If a portion of the metrics have changed, then the objects related to those 
metrics would be new instances, while the other unchanged metrics would be same-instance identical.

A downside to this approach is that authors wouldn't necessarily be able to depend on getting back
the same object identity all the time. For example, JavaScript properties added to a line object might
"stick" on that object only as long as the same instance is returned from the API. Once a "new" object
is returned, the author's extra JavaScript properties will be missing.

Our recommendation is a hybrid approach. New objects shall be created every time `format` is called.
This allows us to provide clear author expectations. However, to allow implementations to optimize, 
only **one copy** of the metrics objects (the one most recently returned from `format`) will be 
"operable" at any one time. Prior copies of metrics objects will be internally disabled such that API
calls on them will throw exceptions.

## Thinking ahead: future integration into DOM or Houdini Layout API

⚠🚧 WARNING: This section is entirely speculative, and out of scope for now. We include it here 
to ponder extended use cases in which these metrics could be applicable in the wider web platform.
(And not to lose track of them in the design process.)

The opportunity to get detailed metrics for formatted text is not exclusively tied to scenarios 
where DOM is potentially unavailable or impractical to use. We would like to ensure that we design
for the possibility of integration into both DOM and Layout API scenarios as well.

We envision APIs similar to `format()`, that could also extract formatted text metrics. For DOM,
a given `Node` already has a layout (when attached to the tree) that includes a Layout box model,
and so a similar `format` call would not require specifying an inline-size constraint. Instead,
something like `measureFormattedText()` would operate at \[some scope] and return the formatted
text metrics for that scope. A more scoped set of metrics could be returned by extending a similar
existing API [`getClientRects()`](https://drafts.csswg.org/cssom-view/#dom-element-getclientrects)
with line metric information.

In the Layout API, while processing a `layout`, `LayoutFragment` objects can represent a line of text.
In these situations, it might make sense to extend the `LayoutFragment` by combining it with a 
`FormattedTextLine` metrics object. This would provide extra information about the intra-line fragments
and glyph information, potentially allowing advanced positioning of glyphs within a line-layout pass.

| ⚠🚧 Ideas for integration into other parts of the platform | Description |
|---|---|
| myElement.`measureFormattedText`() | Similar to `format`. TBD on scope of how this would work 😊 |
| `extDOMRect`.`textFragments`[`i`] | Alternative DOM integration point that extends `getClientRects()` such that each rectangle gets the `FormattedTextLine` mixin or some such. | 
| `extLayoutFrag`.`textFragments`[`i`] | Array of `FormattedTextFragments` (see equivalent functionality in a `FormattedTextLine` object). |

# Formatted text metrics objects

## Paragraphs - `FormattedTextParagraph`

This is the top-level container returned by formatting a FormattedText object. It provides:

* width/height.
* array of Line objects.
* a coordinate system for its lines (see section below).
* Utility function for getting a position from a character and text run in the data model (position
   objects described below).
* Utility function for getting a position from an x/y coordinate pair (where the x/y coordinates 
   should be relative to the paragraph's coordinate system.

| API | Description |
|---|---|
| .`inlineSize` | Returns the bounding-box size in the inline direction or width for horizontal writing modes (double) |
| .`blockSize` | Returns the bounding-box size in the block direction or height for horizontal writing modes (double) |
| .`lines`[] | An array of `FormattedTextLine` objects |
| .`getPosition`(`textrun`, `offset`) | Given the "source object" (intentionally generic: could be extended to support `Text` in the future), and an offset, returns a `FormattedTextPosition` of the associated glyph (if any). If no glyph for that character, returns `null`. |
| .`getPositionFromPoint`(`x`,`y`,`findNearest`) | For mapping pointer positions into glyphs. `findNearest` might ensure that a `FormattedTextPosition` is always returned regardless of the coordinate value, whereas otherwise, `null` might be returned if not strictly over a glyph. |

### Thoughts on coordinate systems



## Additional Accessibility Considerations for Metrics

**Word bounds/breaks** - meta-data about "word" breaks opportunities are not exposed to the developer
today. In the current propsoal, the identification of word breaks is left as an activity to be
supported by author code. Note that word-breaking for the purpose of line wrapping and formatting
is in scope for this feature. However once the layout has been calculated, author code will need 
to use heuristics in langugues that have natural word breaks (e.g., via spaces between words). 
Developer feedback on whether native metrics should be support for word-breaks is sought and may
motivate additional work.

## Contributors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
