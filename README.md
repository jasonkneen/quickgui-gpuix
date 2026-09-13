# quickgui-gpuix

Experimental native-backend adapter for GPUIX 0.7.0 React applications.
QuickGUI owns native layout, input, text, drawing, and application services.
Application components and reactivity remain in React.

This package translates GPUIX committed mutations into QuickGUI native nodes. It is not a
second GPU renderer and does not yet implement the complete GPUIX native interface.
Build with `gpuixQuickGuiPlugin()` so runtime imports of `@gpuix/native` resolve to this adapter.
Use the QuickGUI main-thread host/worker lifecycle; do not launch the GPUIX frame loop.

Run `bun install`, `bun test --preload ./test/preload.ts ./test`, and `bun run typecheck` here.

This is an independent experimental adapter, not an official QuickGUI package.
It currently requires the `experiment/fx-ui-gpuix` branch in sibling `../quickgui`;
upstream main does not yet provide all of the native host capabilities it uses.
The sibling `../fx-ui-quickgui` app consumes this package as a local workspace.

Extracted from `jasonkneen/quickgui` commit `e61b783`. The adapter tests reuse
the sibling SDK fake binding; this checkout contains the adapter and GPUIX patch only.
Native document/editor work remains in the SDK fork pending upstream support.
