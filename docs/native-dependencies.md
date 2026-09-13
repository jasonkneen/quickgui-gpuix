# Native dependency audit

Audited against upstream main at c6bca9fc51e9315942939ddf6aee97d1a89fec44 and fork commit c9a755dd5e28a9399abf6bdb373c7adb15a207ae.
`quickgui` branch `sdk/native-host-capabilities` isolates the native changes in one
commit on upstream main. Its tracked tree is identical to the working fork; no
adapter, application, or package patches are included. Generated Go/TypeScript
protocol definitions and Solid style types accompany the Rust ABI changes.
This is an extraction branch, not a claim that every feature is ready to merge.

## What is actually required?

A GPUIX adapter does not inherently require all these changes. The existing core
already supplies native elements, layout, text, input, windows, lists, anchors and
animations. Our current adapter nevertheless requires the fork: `init()` sends
LayoutRounding (370), every style application sends BlockPointer (365), and
upstream host validation rejects property codes above 360. Replacing only the
library or generated bindings is not a supported way to remove this dependency.

| Group | Current adapter use | Could an upstream-only adapter omit it? |
| --- | --- | --- |
| Host declarations: Motion 361, AnchoredLayer 362, ScrollRequest 367 | Animation declarations, floating layers, programmatic scrolling | Yes for a reduced subset; preserving those behaviors requires exposing equivalent core APIs or implementing a different translation. Current implementation has no fallback. |
| InputSubmitOnEnter 366, BlockPointer 365 | Composer submission and pointer occlusion | Existing inputs/clicks work without these additions, but these semantics need replacement. BlockPointer is occlusion, not a complete CSS pointer-events implementation. |
| InputPresentation 363, OverscanPixels 364, LayoutRounding 370 | Caret/insets, pixel-based list overscan, fractional layout | Upstream defaults can support a functional UI after adapter changes, but would not preserve the current sizing and visual contract. |
| RichDocument 368 and DocumentTheme 369 | Native code/diff documents and custom Markdown theme | Not necessary for generic views or ordinary text. Current fx-ui code/diff/tool content uses them. Replacing with simpler components or future upstream support is possible work, not implemented parity. |
| Text layout, CoreText, glyph rasterization, SVG supersampling, gradients and shadow ordering | Whitespace alignment, text metrics, icon strokes, gradient arithmetic, overlapping shadows | Not necessary to translate React mutations. They were introduced for GPUIX visual parity; omitting them changes pixels and sometimes wrapping/hit geometry. |
| Text intrinsic rounding | Ordinary label measurement | Independent correctness proposal in PR #18; not a prerequisite to the adapter architecture. |

## Source boundaries for subsequent small patches

- Host declarations: `crates/quickgui-host/src/{motion,anchored_layer,scroll_request,input_presentation}.rs`;
  wired through `lib.rs`, `tree.rs`, `view.rs`, and `runtime.rs`. Animation and anchor
  primitives already exist upstream; much of this is host exposure and semantic translation.
- Input/list/layout mechanisms: `src/element/`, `src/text_input.rs`, `src/virtual_list.rs`,
  `src/ui_tree/`, runtime keyboard dispatch and matching host mappings/tests.
- Documents: `src/document/`, Markdown additions, host `document.rs`, syntax dependencies,
  and pointer/selection/link integration. These are primarily viewers and document rendering,
  not a claim to provide a complete source-code editor. Keep separate from the maintainer's editor work.
- Rendering parity: `src/renderer*`, `src/quad.wgsl`, `src/path.rs`, `src/scene.rs`,
  `src/svg_renderer.rs`, vendored cosmic-text/glyphon changes, and Metal generation in `build.rs`.
  Shared files also contain document and layout work; whole-file cherry-picks are not independent patches.
- Protocol: keep Rust property constants and generated bindings together. Removing a group
  needs a deliberate property allocation/generation plan; do not renumber a live host/client pair separately.

## Review status and limits

The audit traces source dependencies; it does not establish that an upstream-only
adapter passes tests. No feature was removed from the working app and no visual
compromise was applied. The extraction branch has the same tracked source tree as
the previously tested fork, so this extraction itself introduces no native changes.

PR #15 review findings still need triage before pitching these groups upstream:
motion initial-only values, scroll revision after list recreation, diff scrolling,
ordinary scroll offset sign, document source aliases, pointer-disabled document links,
gradient alpha dithering and fractional anchor margins. Pixel parity with GPUIX alone
does not prove each behavior correct. The libfx pagination finding belongs to the app
patch, not the SDK extraction.

Recommended submission order: small independently proven fixes; host access to existing
anchor/scroll/input/animation primitives; document support only after coordination;
parity rendering changes with explicit visual and performance evidence. Submitted independently against upstream main:

- [#18](https://github.com/egoist/quickgui/pull/18): intrinsic text rounding.
- [#19](https://github.com/egoist/quickgui/pull/19): opt-in fractional root layout, 1,060 core tests passed.
- [#20](https://github.com/egoist/quickgui/pull/20): variable-list pixel overscan, known heights, and tail-follow correction, 1,061 core tests passed.

Each is ready for review; none is claimed as accepted. The host declaration layer,
input changes, document rendering, and remaining visual parity work are still fork-only.
Their review findings and API boundaries need resolution before further submissions.
