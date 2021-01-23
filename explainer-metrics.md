Formatted Text - Metrics
=============
A representation of post-layout metrics for inline layout content, in particular the
metrics from the formatted text data model.

This explainer focuses on the metrics that are offered for inline text content, in particular
the data model for formatted text. We aspire to create these metrics in a way that allows them
to be supported for other sources of inline text content, in particular the DOM or a Worklet's
Layout API algorithm. For a general overview of the feature, see the repo's [readme](README.md).
You can also learn more about the [formatted text data model](explainer-datamodel.md) and 
[how to render it](explainer-rendering.md).

## Editing scenarios for inline text

Many of the scenarios behind the chosen metrics are based on common editing use cases. An editing
surface must provide a visual view and the means to move insertion points, selection ranges, etc.,
by responding to various input including pointing devices and keyboard. In order to support these
input modalities, the metrics supplied by the explainer chiefly provide the means of understanding
the relationships between parts of text as it was laid out (the glyphs that make up words, lines, sentences,
etc.) and between laid out text and its source objects.

In this explainer we explore these various use cases and build out the set of metrics necessary to
support them.

## Caret Navigation

TBD




## Accessibility Considerations for Metrics

Regarding the metrics needed to make text fully accessible
to an AT (which may become future requirements for canvas) we envision the following needs
(partially met by this proposal):

Note: the following needs updates:

* Word bounds/breaks - used to support "navigate by word" AT features. Break opportunities
  (in HTML) are calculated as part of layout, and as part of the `fillFormattedText`,
  `strokeFormattedText` and `measureFormattedText` APIs to find where line breaks will be
  possible. However, the meta-data about these break opportunities are not exposed to the
  developer.
* Character bounds - used by ATs for character-by-character navigation. The
  `CanvasFormattedTextLineSegment` surfaces `TextMetrics` that will include an `advances`
  array of positions used to describe character bounds.
* Format boundaries - provide opportunities for the AT to optionally add emphasis or
  pass over certain runs of text. The developer has already separated ranges of
  similarly-formatted runs of text into `CanvasFormattedTextRun`s which can be iterated at
  any time to calculate the offset positions to meet this requirement.

We are interested in hearing about additional community feedback related to accessibility
and thoughts on the related open issues.

## Authors:
 [sushraja-msft](https://github.com/sushraja-msft),
 [travisleithead](https://github.com/travisleithead)
