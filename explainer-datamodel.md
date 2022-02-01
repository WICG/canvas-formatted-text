Formatted Text - Input Data Model
=============
The input object model that describes multi-line formatted text before it is shaped and formatted.
The input consists of text (from JavaScript strings), text metadata (e.g., for internationalization),
and text formats/style. Formatted text scenarios involve both document and non-document (Worker)
scenarios and thus have no dependency on DOM Text nodes or Elements.

This explainer focuses on the input data model for formatted text. The output is captured in 
a separate explainer for [formatted text metrics](explainer-metrics.md). The metrics can also
be [rendered](explainer-rendering.md) if desired. For a general overview of the problem space,
see the [readme](README.md).

## Providing an input model for formatted text

On the web today, to render text there are two general options: use the DOM (HTML's 
`CharacterData`/`Text` nodes or SVG Text-related Elements), where text is collected in the
document's "retained" mode infrastructure; the web platform decides when and how to compose and
render the text with declarative input from you (in the form of CSS); or you can use Canvas and
"write" the text when and how you want with JavaScript (an "immediate" mode approach). Canvas
provides very limited text support today and (by design) leaves any special formatting, text
wrapping, international support, etc., up to the JavaScript author.

Of principle interest to most text-based scenarios is not the data model itself--that is,
*applications will already have an existing data model*, and because these applications interface
with the web, they use JavaScript primitives at some point to express their text. A secondary 
"retained" data model specified just for this feature is unnecessary. Instead of creating platform
objects to retain a data model for incremental updates, this feature only requries that existing
JavaScript strings be structured and input into a "formatter" operation in a particular structure 
in order to get to what is really important: [the output text metrics](explainer-metrics.md).

In this explainer, the input structure for formatting text is presented along with a simple structure
for creating reusable formatting.

### Principles
* Enable re-use of existing appliction string data.
* Scope formatting to the needs of inline text layout.
* Leverage CSS as the universal text layout system for the web (even in an imperative model) which 
    also provides for future extensibility.
* Avoid multi-level hierarchal text structures (which ultimately must be linearly iterated for 
    rendering)
* Object-centric model (versus range+indexes) to improve encapsulation and avoid problems with 
    overlapping formatting.
* Extensibility for internationalization hints and reserved future scenarios.

### Formatting

The data model comprises the parameters to the operation `format` which takes the input data model
and produces output [text metrics](explainer-metrics.md). The input consists of:

1. text or an array of text
2. metadata about all the text
3. constraints for the formatting process

Each of these is explained in more detail below.

#### Expressing Text

Text can be expressed as JavaScript strings, or as a JavaScript object with property named
`text` that has a JavaScript string as its value.

```js
// The following representations of text are equivalent:
format( "hello" );
format( { text: "hello" } );
```

Text can also be expressed in separate runs (which are concatenated together when formatted)
when wrapped in an Array. The following are equivalent:

```js
// The following representations of text are equivalent:
format( [ "hello", " world!" ] );
format( [ { text: "hello" }, { text: " world!" } ] );
```

#### Adding metadata

Metadata for all the text can be added 


####

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
text.textruns.push( new FormattedTextRun( " world" ) );
// make a copy of all the text runs from another FormattedText object to be included in a new one:
let text2 = new FormattedText( text.textruns );
// copy everything from another FormattedText object and its text runs
let text3 = new FormattedText( text2 );
```

## CSS Styling of FormattedText and text runs
A wide range of inline layout-related CSS is supported on both the `FormattedText` and
`FormattedTextRun` objects.

To set styles use the `styleMap` property (just like in HTML):

```js
textrun2.styleMap.set("text-decoration", "underline");
```

The `styleMap` property is available on both the `FormattedText` and `FormattedTextRun` objects.

It is also possible to set multiple styles at once similar to how HTML parses the `style` attribute.
Set styles all at once at construction time:

```js
let textrun2 = new FormattedTextRun( { style: "color: yellow; font: 15pt Verdana" } );
```

Or set groups of styles together dynamically afterward:

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

The above is the semantic equivalent to the following declarative HTML markup:

```html
<div>
  The quick <span style="color: brown; font-weight: bold">brown</span> fox jumps over the lazy dog
</div>
```

Where the `<div>` element is the container for an inline formatting context (and can be styled), 
and the `<span>` contains the formatting for the word "brown". More precisely, since each 
`FormattedTextRun` object has the potential to be styled, the following markup better represents 
the semantic equivalent:

```html
<div>
  <span>The quick </span>
  <span style="color: brown; font-weight: bold">brown</span>
  <span> fox jumps over the lazy dog</span>
</div>
```

## CSS to achieve advanced scenarios

### Vertical Text

By leveraging existing CSS styles for writing modes and other related properties, the 
formatted text data model can support a variety of vertical text scenarios. For 
example, by simply leveraging the `writing-mode` property on the `FormattedText` 
object, we can orient text in its traditional vertical direction:

```js
let proverb = new FormattedText( "不怕慢，", "就怕站" );
proverb.styleMap.set( "writing-mode", "vertical-rl" );
proverb.textruns[1].styleMap.set( "font-weight", "bold" );
```

Together with a nice font and constrained vertical layout bounds, this would render as:

<img src="explainerresources/vertical-text-cn.png" alt="Characters of an ancient Chineese proverb, vertically oriented in two columns, the second column bold">

In combination with other related CSS properties, many other vertical text layouts are
possible:

```js
let proverbEn = new FormattedText( "It's better to make slow progress", " than no progress at all" );
proverbEn.styleMap.set( "writing-mode", "vertical-lr" );
proverbEn.styleMap.set( "text-orientation", "upright" );
proverbEn.styleMap.set( "line-height", "2" );
proverbEn.styleMap.set( "text-align", "center" );
proverbEn.textruns[1].styleMap.set( "font-weight", "bold" );
```

When rendered in an inline-direction constrained space, this would render as:

<img src="explainerresources/vertical-text-en.png" alt="The text 'It's better to make slow progress than no progress at all' rendered vertically from left-to-right in five columns">

### How much CSS should be supported?

`FormattedText` and `FormattedTextRun` objects support various CSS properties that 
influence how the text's lines will ultimately be positioned. There are also many
CSS properties that do not apply to text, that convert between typical text layout
and other layouts, or that take normal flow content out of flow (e.g., `float`, `position`, 
`display`, etc.). For the purposes of a formatted text object model, not all CSS properties
can or should be supported. The guidelines for what CSS to support and what not to support 
follow.

#### Focus on text-related CSS properties

We believe it makes sense to only support CSS properties that provide specific features
for inline-level content (text) and the management of the text's container. Other properties,
especially those that would change the layout characteristics of `FormattedTextRun`
objects from their assumed inline-level nature, will not be supported. In this data model
we want to keep the semantics of a `FormattedTextRun` consistent with the CSS that can be
applied.

Applying the above principle to `FormattedTextRun` objects means that `float` would not
be supported because it has the effect of changing the object's computed display value 
from inline to block (taking it out-of-flow). `position: absolute` also takes
the object out of the normal flow. Likewise, the `width` and `height` properties would
be ignored because they do not apply to inline-level elements. Attempting to set these 
properties would have no effect. On the other hand, `padding` would be supported and 
applied to `FormattedTextRun` objects in accordance with the rules of CSS inline layout,
e.g., the inline-direction `padding` values are taken into account while the 
block-direction values are not.

The `FormattedText` object is an independent containing block for text. It is essentially a 
`display: flow-root` object that establishes an inline formatting context for its children.
The `FormattedText` object has no positional relationship with anything other than its
children, and so values like `top`, `right`, `bottom`, and `left` do not apply (i.e., the 
object acts as its own initial containing block). Positioning the object for the purpose of
rendering in some context must be done separately. In order to render or extract useful metrics
from the `FormattedText` object, final dimensions for its size must be supplied (see 
[Rendering explainer](explainer-rendering.md)) which are used to resolve relative values in
supported properties like `width` and `height`.

In some cases, we imagine it will be useful to allow the `FormattedText` object to assume
alternate layout container types where those alternate types provide unique text layout
capabilities. For example, we expect to support `FormattedText` objects with an inner 
display type of `ruby` in order to become a Ruby container and enable the use of Ruby
annotated layout. Other container types are not currently planned to initially support 
but are good long-term candidates (e.g., multi-column containers created via the `columns`
shorthand property), while still others are less-likely to be supported (e.g., flex and 
grid container types, which are less useful for formatted text scenarios).

There are various CSS properties that provide helpful graphical emphasis to text that are 
also supported. These are for convenience in supporting common text formatting scenarios
that would otherwise require detailed introspection of the object model's related metrics
in order to correctly layout and render as desired with respect to the text. Because these
features are already available in CSS layout engines and significantly ease author burden, 
many of these CSS properties will be supported. Some supported examples include: 
`text-decoration`, `text-shadow`, `box-shadow`, even `border`, `outline`, and limited `background` support 
(where the metrics and composition processing do not require external dependencies, such as 
image resources typically loaded by `url()` functions).

#### Future extensions 

By leveraging CSS, we get the added benefit of a well-known constraint language for 
expressing box bounds and the expected behavior for content (in our case formatted text 
content) that overflows those bounds. `width` and `height` and corresponding `min-width`
or `max-height` properties express the desired constraints for eventual line-box 
computation when it comes to rendering the object model or returning metrics. Similarly,
`overflow` and `clip-path` can further ensure the text content expands or is clipped to
fit the desired constraints. These properties would only apply on the `FormattedText` 
container object.

CSS continues to evolve, and it makes sense to extend relevant new CSS properties to
this object model as they become a part of the web platform. For example, while not
widely supported at the time of writing, support for `shape-inside` (CSS Shapes L2)
and CSS Exclusions provide exciting growth opportunities for text using this model.

## WebIDL

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
  DOMString style = "";
}; 

[Exposed=Window,Worker] 
interface FormattedTextRun { 
  constructor();
  constructor(DOMString text);
  constructor(FormattedTextRunInit textRunInit);

  attribute DOMString text;

  // owning FormattedText (if any) 
  readonly attribute FormattedText? parent;        // null if not contained in a FormattedText
 
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

We've compiled a list of text-related (or generally applicable) CSS properties that we believe make
sense to support on `FormattedText` and `FormattedTextRun` objects. This list is not exhaustive.
For example, it does not include many of the new logical properites such as `inline-size` for 
`width`. This list is provided for potential testing purposes and to facilitate discussion.

| CSS Property | FormattedText | FormattedTextRun | inherits | Notes |
|--------------|---------------|------------------|----------|-------|
| background | ✔ | ✔ | no | background-attachment, background-origin, background-position, background-repeat, background-size operate on an external resource image and will be ignored. Background-image will only support `<gradient>` functions |
| border | ✔ | ✔ | no |  |
| border-image | ✔ | ✔ | no | border-image-source will only supports `<gradient>` functions |
| border-radius | ✔ | ✔ | no |  |
| box-decoration-break | ✔ | ✔ | no |  |
| box-shadow | ✔ | ✔ | no |  |
| box-sizing | ✔ |  | no |  |
| clip-path | ✔ | ✔ | no |  |
| color | ✔ | ✔ | yes |  |
| direction | ✔ | ✔ | yes |  |
| display | ✔ | ✔ | no | Generally not supported, but may make exceptions, e.g., ruby |
| font | ✔ | ✔ | yes |  |
| font-feature-settings | ✔ | ✔ | yes |  |
| font-kerning | ✔ | ✔ | yes |  |
| font-size-adjust | ✔ | ✔ | yes |  |
| height | ✔ |  | no |  |
| hyphens | ✔ | ✔ | yes |  |
| letter-spacing | ✔ | ✔ | yes |  |
| line-break | ✔ | ✔ | yes |  |
| line-height | ✔ | ✔ | yes |  |
| margin | ✔ | ✔ | no |  |
| mask | ✔ | ✔ | no | mask-border-source will only supports `<gradient>` functions |
| mask-border | ✔ | ✔ | no |  |
| max-height | ✔ |  | no |  |
| max-width | ✔ |  | no |  |
| min-height | ✔ |  | no |  |
| min-width | ✔ |  | no |  |
| opacity | ✔ | ✔ | no |  |
| outline | ✔ | ✔ | no |  |
| overflow-wrap | ✔ | ✔ | yes |  |
| padding | ✔ | ✔ | no |  |
| tab-size | ✔ | ✔ | yes |  |
| text-align | ✔ |  | yes |  |
| text-align-all | ✔ |  | yes |  |
| text-align-last | ✔ |  | yes |  |
| text-combine-upright | ✔ | ✔ | yes |  |
| text-decoration | ✔ | ✔ | no | |
| text-emphasis | ✔ | ✔ | yes | |
| text-indent | ✔ |  | yes |  |
| text-justify | ✔ | ✔ | yes |  |
| text-orientation | ✔ | ✔ | yes |  |
| text-overflow | ✔ |  | no |  |
| text-shadow | ✔ | ✔ | yes |  |
| text-transform | ✔ | ✔ | yes |  |
| text-underline-offset | ✔ | ✔ | yes |  |
| text-underline-position | ✔ | ✔ | yes |  |
| transform | ✔ |  | no |  |
| transform-box | ✔ |  | no |  |
| transform-origin | ✔ |  | no |  |
| unicode-bidi | ✔ | ✔ | no |  |
| white-space | ✔ | ✔ | yes |  |
| width | ✔ |  | no |  |
| word-break | ✔ | ✔ | yes |  |
| word-spacing | ✔ | ✔ | yes |  |
| word-wrap | ✔ | ✔ | yes |  |
| writing-mode | ✔ | ✔ | yes |  |

### Limitations

* **CSS pseudo-elements** (`::first-letter`, `::first-line`, `::before`, `::after`). Pseudo-elements
   would require some unique way to specify the style map for these. A workaround for `::first-letter` 
   is relatively easy (separating it into a separate `FormattedTextRun`, or implementing support for
   the related property `initial-letter`), but `::first-line` is harder to target given it depends on 
   where the line breaks.
*  explicit values of `inherit` on CSS properties that aren't specified to inherit by default will not
   likely be supported.

## Accessibility Considerations

Because the `FormattedText` object is a retained data model for text, it has the potential to be a 
fully accessible object (except it has no "view"--which might be an advantage in many scenarios).
We are still thinking about what it would mean to make formatted text accessible and whether it makes
sense. We welcome your comments and issues in this regard.

## Alternatives and Prior Art

The formatted text object model is designed to be very high-level, combining a simple linear text-run
model with the familiarity of CSS. We recognize that one of the significant gaps in the platform is the
lack of ability to understand how text was ultimately formatted. This information is critical in many
scenarios where web developers want to take more control over text layout. While not detailed in this
document, we do plan to work on a detailed metrics API that we aspire to apply to both this object model
as well as in CSS Houdini's Layout API and even to regular DOM elements hosting inline formatting contexts.

As we move into investigating metrics, we intend to work closely with the proponents of the 
[Text Shaping API](https://docs.google.com/document/d/1hBHETpotl4cnvN1dDts_gb3hGTF70_DxjCcERoDrX-A/edit?ts=602d763a#heading=h.xwhbmm78cnfj)
to be sure we are addressing relevant use cases.

The SKIA TextBlob and [TextBlobBuilder](https://api.skia.org/classSkTextBlobBuilder.html) provide a related API for shaping text within a bounds.

## Rendering the FormattedText
The [next explainer](explainer-rendering.md) describes how to take the data model representation of
text and make is show up on screen.

## Contributors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
