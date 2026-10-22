# RFC: stable headless C ABI for darktable

Status: **proposal only**. No ABI symbols are implemented or exported by this change.

## Motivation

Allow desktop, mobile, and CLI frontends to embed darktable's existing RAW decoder, IOP modules, pixelpipe, edit history, and sidecar handling without exposing GTK or internal C structs as a binary interface.

## Proposed contract

- A versioned, C-callable ABI with opaque engine, image, module-instance, mask, and render handles. Explicit capability/version negotiation, typed errors, allocator ownership, and thread-affinity rules.
- A headless engine lifecycle backed by a transient SQLite `:memory:` catalog; explicit image open/unload/reload, resource limits, and no persistent catalog requirement.
- Discover available IOPs and instantiate multiple instances, enumerate/edit typed parameters, enable/disable, reorder, transact edits, and capture snapshots. Internal module parameter blobs must not become ABI structs.
- Specialized geometry and native mask operations for interactions not expressible as scalar IOP parameters (including liquify control points). Use stable opaque IDs, not pointers or database row IDs.
- Render preview/full resolution to caller-owned buffers with explicit pixel format, color profile, stride, ROI, cancellation, and error reporting.
- Import/export XMP using darktable's own serializers. Treat XMP and companion `.dtdata` pixel-backed mask assets as a single logical edit package, preserving forward compatibility and unknown assets.
- Support Linux/macOS/Windows library builds as an eventual target; iOS packaging is a separate feasibility milestone, not a promise of this RFC.

## Compatibility and implementation boundaries

The public ABI must not expose GTK, GLib, SQLite, `dt_develop_t`, `dt_iop_module_t`, pixelpipe structs, or plugin-specific parameter layouts. Separate ABI versioning from internal module serialization versions. Define ownership, locking, callbacks, and whether more than one engine can exist in a process before implementation.

Existing `darktable-cli` already exercises export without a GUI. The implementation should reuse its established startup and processing paths, rather than stubbing GTK symbols or duplicating the image engine.

See `dev-doc/Core_Concepts.md`, `dev-doc/IOP_Module_API.md`, `dev-doc/pixelpipe_architecture.md`, `dev-doc/introspection.md`, and `dev-doc/dtdata_sidecar.md` for existing implementation constraints.

## Suggested milestones

1. Agree on ABI lifecycle, memory ownership, versioning, and error semantics.
2. Implement headless initialization and a single-image in-memory catalog.
3. Implement IOP discovery, instance management, and parameter introspection.
4. Implement pixel rendering with regression tests.
5. Add native masks and geometry operations.
6. Add XMP + `.dtdata` round-trip tests, memory-lifecycle tests, and platform packaging.

## Validation plan

- Compile/link ABI smoke tests on supported desktop targets.
- Test repeated open/render/unload and memory behavior.
- Compare rendering against `darktable-cli` with integration pixel tolerances.
- Test IOP multi-instance history replay, native masks, XMP + `.dtdata` round trips.
- Run feature-on/feature-off builds, plus sanitizer checks.

## Open questions for maintainers

1. Should darktable maintain an embedded engine ABI upstream?
2. Which current headless CLI/export initialization path is the preferred extraction point?
3. What subset of IOP and mask operations can be supported long term?
4. How should XMP and `.dtdata` be packaged and versioned?
5. What platforms and thread-safety guarantees are realistic?

This is an RFC, **not a working SDK**, and should not be merged as an implementation. Prepared with AI assistance; maintainer review is requested before implementation.