Formatted Text - Metrics
=============
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

### Use case: `FormattedText` paragraph placement

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

### Use Case: `FormattedText` line placement

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

Author provides same information as in the previous use case. 

Metrics provide: 

* List of Shaped Glyph metrics per fragment (fragment is a unit of glyphs that all share the
   same format/font/bidi/etc.) such as position, advance, bearing, bounding box, kerning 
   between glyphs. 
* Position references of each glyph's extent in the input characters (of the FormattedText object). 

A rendering API must provide:

* (x, y) location to place a glyph, given a Fragment, and glyph index (within that fragment)

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
