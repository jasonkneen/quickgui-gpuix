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
It currently requires the `sdk/native-host-capabilities` branch in sibling `../quickgui`;
upstream main does not yet provide all of the native host capabilities it uses.
The sibling `../fx-ui-quickgui` app consumes this package as a local workspace.

Extracted from `jasonkneen/quickgui` commit `e61b783`. The adapter tests reuse
the sibling SDK fake binding; this checkout contains the adapter and GPUIX patch only.
Native document/editor work remains in the SDK fork pending upstream support.

See [native dependency audit](docs/native-dependencies.md) for required capabilities,
optional parity work, and the isolated SDK branch.

## Get started from source

Clone these repositories beside each other:

```sh
git clone --branch sdk/native-host-capabilities https://github.com/jasonkneen/quickgui.git
git clone https://github.com/jasonkneen/quickgui-gpuix.git
cd quickgui
bun install
cd ../quickgui-gpuix
bun install --frozen-lockfile
bun run test
bun run typecheck
```

The SDK baseline is commit `804247663f79cb1d89876589fd70ccef9b2fe059`.
Build the SDK's native library following its TypeScript guide before running an app.
The adapter is distributed here as source, not as an npm release. Its local workspace
references deliberately select the fork; installing published upstream QuickGUI
packages does not supply the extra host properties.

An application workspace includes `../quickgui-gpuix`, `../quickgui/packages/native`
and `../quickgui/packages/solid`, and depends on `quickgui-gpuix` via `workspace:*`.
Copy the GPUIX `patchedDependencies` entry and patch file into the application root;
Bun applies dependency patches at the installing workspace root.
Add `gpuixQuickGuiPlugin()` from `quickgui-gpuix/build` to `Bun.build` plugins.
Create `GpuixRenderer` from `quickgui-gpuix`, pass it to GPUIX `createRoot`, and
forward its callback to GPUIX `handleGpuixEvent`. Run application initialization
through QuickGUI's worker lifecycle, with `runHost` on the main thread.

macOS is the exercised native runtime. Other platforms and the complete GPUIX API
remain unverified. See the dependency audit for known review findings and limitations.
