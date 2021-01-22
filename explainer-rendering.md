Formatted Text - Rendering
=============
An approach to rendering multi-line formatted text to an HTML canvas context.

This explainer focuses on rendering [the data model](explainer-datamodel.md) for formatted text.
For a general overview of the problem space, see the repo's [README.md]. In addition to rendering
the data model can [provide metrics](explainer-metrics.md).


---

Next, to render the text into multiple lines, we use the new `fillFormattedText` API
which takes a max width to wrap the text at.

```js
context.font = "18pt Arial";
context.fillFormattedText( formattedText, /*x*/0, /*y*/50, /*wrapWidth*/250 );
```

This would produce the following output on the canvas:

<img src="explainerresources/Example1.png" alt="Wrapped text rendered in a canvas." align="center"/>

### Bidi Text

No additional work is needed from web developers to support bidi text.
Implementations will perform bidi analysis on the `CanvasFormattedText`'s text runs
and create internal bidi runs if necessary. An example demonstrating bidi text follows:

```js
const context = document.getElementById( "myCanvas" ).getContext( "2d" );
context.font = "30px Arial";
let canvasFormattedText = new CanvasFormattedText();
canvasFormattedText.appendRun( { text: "Sample arabic بمدينة مَايِنْتْس، ألمانيا text." } );
context.fillFormattedText( canvasFormattedText, /*x*/0, /*y*/30, /*wrapWidth*/350 );
```

produces the following output on the canvas:

<img src="explainerresources/Example2.png" alt="Wrapped text rendered in a canvas." align="center"/>

### Pre-existing text controls

The other text styles `textAlign`, `textBaseline` on the canvas context control justification
and baseline alignment of the multiline text relative to provided x/y coordinates.

CSS styles on the canvas element affect text rendering. These CSS properties are

- `line-height` - Specifies height of a line.
- `direction` - Sets the initial direction for bidi analysis.
- `word-break` / `word-wrap` - Controls break opportunities for text wrapping.


## Advanced Usage

While the single shot `fillFormattedText` and `strokeFormattedText` APIs gets us a
long way in adding text wrapping support, it is also conceivable that developers will
want to render text one line at a time. This offers advanced control over both position
and available width used for each line. Additionally, rendering text one line at a time
also provides developers the flexibility to render only the content visible onscreen or
content that changed.

To render one line at a time, we need to introduce a few additional objects and concepts.
First of all, we need to be able to indicate what portion of the aggregate `CanvasFormattedText`
text runs we need to render on each separate line, and we need to be able to specify a maxium
width to render them into (which can change from line to line). The API is designed to allow
an iterative approach to line rendering, but is also flexible enough to allow for many other
scenarios.

### Position objects

In both bidi and regular text scenarios lines always contain a contiguous range
of characters from the source text. The position at which to start the next line
can be represented with an offset in that text.

To indicate a position to begin rendering a text run, we define a position object that has three
components:
1. `formattedText` - required `CanvasFormattedText` reference - the container to which this
    position object belongs.
2. `textRunIndex` - required number - the index of a text run object contained by the
    `CanvasFormattedText` (e.g., the object associated with the text run "the quick " in
    the example above.
3. `textRunOffset` - required number - the offset into the text run value itself. For example:
    a `2` would indicate a position starting at the "e" in the string "the quick".

For convenience, one of these objects (a `CanvasFormattedTextPosition` dictionary) can be obtained
from the `CanvasFormattedText` using the `beginPosition()` function.

We use the position object to indicate the starting position within the `CanvasFormattedText`
object from which we'd like to start rendering a line. But, where should the ending position be?
Rather than try to guess or iteratively test, we can have the API figure this out for us. All it
needs to know is how much space we'd like to allocate for the line.

### CanvasFormattedTextLine object

The desired width of the line and the starting position is provided to the `measureFormattedText`
function, a new formatted text measurement API on the canvas context. It returns a
`CanvasFormattedTextLine` object that is ready for rendering and contains some other helpful
information packed into it. To render the line to the canvas we introduce `fillFormattedTextLine`
and `strokeFormattedTextLine` which conveniently take a `CanvasFormattedTextLine` object and an x/y
coordinate where the text should be rendered to the canvas. The `CanvasFormattedTextLine` keeps the
line state (such as the desired width) so that only the location where the line should be rendered
is needed when it comes time to draw.

Each `CanvasFormattedTextLine` provides a `nextPosition()` function that returns a position object
for where the next line should start from in the case that there is more text to render from the
`CanvasFormattedText` object. If the line represents the last line of text to render, then
`nextPosition` returns `null`.

Here's an example that shows how these objects relate to each other, illustrating the use of
`fillFormattedTextLine` to render each line of our first example.

```js
const context = document.getElementById("myCanvas").getContext("2d");

// Collect text into a CanvasFormattedText object
let formattedText = new CanvasFormattedText();
formattedText.appendRun( { text: "the quick " } );
formattedText.appendRun( { text: "brown", font: `bold ${context.font}` } );
formattedText.appendRun( { text: " fox jumps over the lazy dog" } );

let startPosition = formattedText.beginPosition();
while ( startPosition ) {
  let currentLineObject = context.measureFormattedText( startPosition, /*wrapWidth*/250 );
  context.fillFormattedTextLine( currentLineObject, /*x*/0, y );
  startPosition = currentLineObject.nextPosition();
  y += (currentLineObject.height);
}
```

We can use this additional flexibility to adjust each line's width and position to accomodate
any other objects being presented to the canvas. In this example, we adjust the lines to wrap
around an image.

```js
const context = document.getElementById("target").getContext("2d");
// Grab a previously loaded <img> element to render on the canvas.
const image = document.getElementById("myImg");

let sampleString = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Mi proin sed libero enim sed faucibus turpis. Sed augue lacus viverra vitae. Adipiscing tristique risus nec feugiat. Odio facilisis mauris sit amet massa. Non arcu risus quis varius quam quisque id diam vel. Scelerisque varius morbi enim nunc faucibus a pellentesque sit amet. Lectus proin nibh nisl condimentum. Eget mauris pharetra et ultrices neque ornare aenean euismod elementum. Eget magna fermentum iaculis eu non. Id aliquet risus feugiat in ante. Gravida in fermentum et sollicitudin ac. ";

// Draw the image on the top left of the canvas.
let x = 0, y = 0;
// Define margins around the image.
let topLeftMargin = 10, bottomRightMargin = 10;
context.drawImage( image, topLeftMargin + x, topLeftMargin + y );

// Setup the formatted text object.
let formattedText = new CanvasFormattedText();
formattedText.appendRun( { text: sampleString } );

const fontHeight = 20;
context.font = `${ fontHeight }px Arial`;

// Draw each line adjusting available width if there is an image on the
// left that reduces the available width.
const maxWidth = context.canvas.width - bottomRightMargin;
y += fontHeight;
let lineStartPosition = formattedText.beginPosition();
do {
  // Determine x position.
  if ( y <= ( topLeftMargin + image.height + bottomRightMargin ) ) {
    x = topLeftMargin + image.width + bottomRightMargin;
  }
  else {
    x = topLeftMargin;
  }
  let line = context.measureFormattedText( lineStartPosition, maxWidth - x );
  context.fillFormattedTextLine( line, x, y );
  y += line.height;
  lineStartPosition = line.nextPosition();
} while ( lineStartPosition );

```

This would produce the following output on the canvas

<img src="explainerresources/Available-Width.png" alt="Example use case for rendering multiline text with varying available width" align="center" style="border:2px solid black;"/>

The separation of measuring to obtain a `CanvasFormattedTextLine` object and the request to
draw the line using `fillFormattedTextLine` or `strokeFormattedTextLine` allows for a
useful scenario in adjusting line separation; `CanvasFormattedTextLine` objects can be collected
and used to sum the total height needed to render all the lines allowing the developer to
distribute the line-spacing as desired.

Additional `TextMetrics` info can be obtained from each `CanvasFormattedTextLine` object by iterating
over the `lineSegments` array. A "line segment" is either:
1. The same as the developer-provided text run, or;
2. Part of the developer-provided text run which can happen when the line wrapped in the middle
   of a text run, or when text with different bidi characteristics are encountered
   (e.g., RTL inside of LTR text).

Line segment objects have 4 properties:
* `beginPosition` - a reference to the text run and offset where this segment starts
* `endPosition` - a reference to the text run and offset where this segment ends
* `isRTL` - a boolean indicating if the segment's text is right-to-left rendered
* `textMetrics` - a reference to the existing canvas `TextMetrics` object for this segment.

Segment information can be helpful when desiring to simulate cursor movement over a line or to
draw backgrounds, underlines, borders, etc., with the combination of the `isRTL` value and
proposed `TextMetrics`'s `advances` (see related issues
[3994](https://github.com/whatwg/html/issues/3994),
[4030](https://github.com/whatwg/html/issues/4030), and
[4034](https://github.com/whatwg/html/issues/4034))

## WebIDL details

```webidl

// Extends the existing CanvasText mixin interface, which is included in:
// * CanvasRenderingContext2D
// * OffscreenCanvasRenderingContext2D
// (notably not included in PaintRenderingContext2D;
//  see :https://drafts.css-houdini.org/css-paint-api-1/#2d-rendering-context)
partial interface CanvasText {
    // Render entire CanvasFormattedText with line wrapping (one-shot)
    void fillFormattedText(CanvasFormattedText formattedText,
                          double x,
                          double y,
                          double wrapWidth);
    void strokeFormattedText(CanvasFormattedText formattedText,
                            double x,
                            double y,
                            double wrapWidth);

    // Line-at-a-time text measurement
    CanvasFormattedTextLine measureFormattedText(CanvasFormattedTextPosition textPosition,
                                                double wrapWidth);

    // Line-at-a-time text rendering (given a previously measured line).
    void fillFormattedTextLine(CanvasFormattedTextLine line,
                              double x,
                              double y);
    void strokeFormattedTextLine(CanvasFormattedTextLine line,
                                double x,
                                double y);
};


[Exposed=Window,Worker]
interface CanvasFormattedText {
  constructor();

  CanvasFormattedTextPosition beginPosition();
  getter CanvasFormattedTextRun getRun(unsigned long index);
  CanvasFormattedTextRun appendRun(CanvasFormattedTextRun newRun);

  readonly attribute unsigned long length;
};


dictionary CanvasFormattedTextRun {
  required DOMString text;
  DOMString font;
  DOMString color;
};


dictionary CanvasFormattedTextPosition {
  required CanvasFormattedText formattedText;
  required unsigned long textRunIndex;
  required unsigned long textRunOffset;
};


// Returned exclusively from measureFormattedText()
[Exposed=Window,Worker]
interface CanvasFormattedTextLine {
  readonly attribute double height;
  readonly attribute double width;
  CanvasFormattedTextPosition? nextPosition();
  readonly attribute FrozenArray<CanvasFormattedTextLineSegment> lineSegments;
};


dictionary CanvasFormattedTextLineSegment {
  required CanvasFormattedTextPosition beginPosition;
  required CanvasFormattedTextPosition endPosition;
  required boolean isRTL;
  required TextMetrics textMetrics;
};
```

## Open issues and questions

Please review and comment on our [existing open issues](https://github.com/MicrosoftEdge/MSEdgeExplainers/issues?q=is%3Aissue+is%3Aopen+label%3A%22Canvas+Formatted+Text%22).

Additionally, here are a few minor tweaks suggested for the current API surface (subject to change):
   * Recommend adding a constructor overload of `CanvasFormattedText` that takes a list
     of `CanvasFormattedTextRun` objects, e.g.,
     `constructor( sequence<CanvasFormattedTextRun> formattedRunsInit );`
   * Recommend renaming 'appendRun' to 'addRun' to match Path2D:addPath() nomenclature;
   * Recommend returning CanvasFormattedText object [this] from 'appendRun' to allow
      chaining calls together...
   * Recommend making `CanvasFormattedText` object an iterable, e.g.,
      `iterable<CanvasFormattedTextRun>;`
   * Recommend dropping the 'getRun' function name (getter is probably good enough)
   * `CanvasFormattedTextRun`:`color` could be named "style" to more closely match
      related stroke/fillStyle. Will patterns and gradients work here too?
   * `CanvasFormattedTextLine` has `width` and `height`, might also benefit from
      additional TextMetrics interface methods: `actualBoundingBoxLeft` (and Right,
      Ascent, Descent) properties.

## Alternatives Considered

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

## Accessibility Considerations

Making the Canvas accessible is a persistant challenge for the web today. Several
efforts are underway, including a
[promising solution](https://github.com/WICG/aom/blob/gh-pages/explainer.md#use-case-4-adding-non-dom-nodes-virtual-nodes-to-the-accessibility-tree)
as part of the Accessible Object Model (AOM) family of proposals.

Meanwhile, web developers are encouraged to use the canvas element's fallback content
to provide HTML markup that describes the canvas' current state. For text used as part
of this API proposal, the complete source text of the `CanvasFormattedText` should be
placed in block-styled
[flow content](https://html.spec.whatwg.org/multipage/dom.html#flow-content-2) (such
as a `<p>` element), with formatted sections wrapped in appropriate
[phrasing content](https://html.spec.whatwg.org/multipage/dom.html#phrasing-content-2)
(such as `<span>` and styled to match the `CanvasFormattedTextRun` formatting.) The
markup should make use of
[ARIA Live Regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)
to be sure assistive technologies (ATs) pickup and announce any dynamic changes.

Looking generally at what low-level features are necessary to make text fully accessible
to an AT (which may become future requirements for canvas) we envision the following needs
(partially met by this proposal):

* Word bounds/breaks - used to support "navigate by word" AT features. Break opportunities
  (in HTML) are calculated as part of layout, and as part of the `fillFormattedText`,
  `strokeFormattedText` and `measureFormattedText` APIs to find where line breaks will be
  possible. However, the meta-data about these break opportunities are not exposed to the
  developer (as noted in open issues).
* Character bounds - used by ATs for character-by-character navigation. The
  `CanvasFormattedTextLineSegment` surfaces `TextMetrics` that will include an `advances`
  array of positions used to describe character bounds.
* Format boundaries - provide opportunities for the AT to optionally add emphasis or
  pass over certain runs of text. The developer has already separated ranges of
  similarly-formatted runs of text into `CanvasFormattedTextRun`s which can be iterated at
  any time to calculate the offset positions to meet this requirement.

We are interested in hearing about additional community feedback related to accessibility
and thoughts on the related open issues.

## Privacy Considerations

HTML5 canvas is a browser finger printing vector
(see [canvas fingerprinting](https://en.wikipedia.org/wiki/Canvas_fingerprinting)).
Fingerprinting happens through APIs `getImageData`, `toDataURL`, etc. that allow
readback of renderer content exposing machine specific rendering artifacts.
This proposal adds the ability to render multiple lines of text, potential
differences in text wrapping across browsers could contribute to additional
fingerprinting. The existing implementer mitigations in some user agents that
prompt users on canvas readback continues to work here.

We are currently evaluating whether this API would increase fingerprinting surface
area and will update this section with our findings. We welcome any community feedback.
