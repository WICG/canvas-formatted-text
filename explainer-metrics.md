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

