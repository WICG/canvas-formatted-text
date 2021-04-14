# 🚧 Variations from the explainer in the Chromium experimental implementation 🚧

## API Shape

1. The explainer anticipates using an `ObservableArray` type for managing the list of text runs in the [data model](explainer-datamodel.md). Until ObserverableArray is implemented, the list of text runs differs in the following ways:
    * `textruns` attribute is not supported
    * `FormattedText`.`appendRun()` is how text runs are added
    * `FormattedText`.`deleteRun()` is how text runs are deleted
    * `FormattedText` has an operable indexed getter (but no setter)
2. The implementation uses an older constructor name `CanvasFormattedText` instead of `Formattedtext`
3. Specified overloads for the constructors are not supported (e.g., `new FormattedText( "run1", "run2" )`, `new FormattedText({text:"run1",style:"color:red"})`, etc.)
4. `setStyle()` is not supported.

## Supported CSS values

Summary of CSS subset in blink/renderer/core/css/css_properties.json5, starting from [commit 2788582](https://chromium-review.googlesource.com/c/chromium/src/+/2788582/14/third_party/blink/renderer/core/css/css_properties.json5)

| CSS Property | FormattedText | FormattedTextRun | inherits | Notes |
|--------------|---------------|------------------|----------|-------|
| background | | | no | Currently only painting the foreground layer |
| border |  |  | no |  |
| border-image |  |  | no | border-image-source will only supports `<gradient>` functions |
| border-radius |  |  | no |  |
| box-decoration-break |  |  | no |  |
| box-shadow |  |  | no |  |
| box-sizing |  |  | no |  |
| clip-path |  |  | no |  |
| color | ✔ | ✔ | yes |  |
| direction | ✔ | ✔ | yes |  |
| display |  |  | no | Generally not supported, but may make exceptions, e.g., ruby |
| font | `font-family`, `font-size`, `font-stretch`, `font-style`, `font-variant-ligatures`, `font-variant-caps`, `font-variant-east-asian`, `font-variant-numeric`, `font-weight`, `font-variant-settings` | same as `FormattedText` column | yes |  |
| font-feature-settings | ✔ | ✔ | yes |  |
| font-kerning | ✔ | ✔ | yes |  |
| font-size-adjust |  |  | yes |  |
| height | ✔ |  | no |  |
| hyphens |  |  | yes |  |
| letter-spacing |  |  | yes |  |
| line-break | ✔ (and `-webkit-` prefix) | ❌ | yes |  |
| line-height |  |  | yes |  |
| margin |  |  | no |  |
| mask |  |  | no | mask-border-source will only supports `<gradient>` functions |
| mask-border |  |  | no |  |
| max-height |  |  | no |  |
| max-width |  |  | no |  |
| min-height |  |  | no |  |
| min-width |  |  | no |  |
| opacity |  |  | no |  |
| outline |  |  | no |  |
| overflow-wrap |  |  | yes |  |
| padding |  |  | no |  |
| tab-size |  |  | yes |  |
| text-align | ✔ |  | yes |  |
| text-align-all |  |  | yes |  |
| text-align-last | ✔ |  | yes |  |
| text-combine-upright |  |  | yes |  |
| text-decoration | `text-decoration-color`, `text-decoration-line`, `text-decoration-skip-ink`, `text-decoration-style`, `text-decoration-thickness` | same as `FormattedText` column | no | |
| text-emphasis |  |  | yes | |
| text-indent | ✔ |  | yes |  |
| text-justify | ✔ | ❌ | yes |  |
| text-orientation | ✔ | ❌ | yes |  |
| text-overflow | ✔ |  | no |  |
| text-shadow | ✔ | ✔ | yes |  |
| text-transform | ✔ | ✔ | yes |  |
| text-underline-offset | ✔ | ✔ | yes |  |
| text-underline-position | ✔ | ✔ | yes |  |
| transform |  |  | no |  |
| transform-box |  |  | no |  |
| transform-origin |  |  | no |  |
| unicode-bidi |  |  | no |  |
| white-space |  |  | yes |  |
| width | ✔ |  | no |  |
| word-break |  |  | yes |  |
| word-spacing |  |  | yes |  |
| word-wrap |  |  | yes |  |
| writing-mode | ✔ | ❌ | yes |  |
