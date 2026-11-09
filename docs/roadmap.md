# Implementation roadmap

The target is a useful Flutter workflow in Zed. Feature parity must be measured against tested behavior and the available extension APIs.

## 1. Establish the existing baseline

- [x] Inspect Flutter, Dart Code, and Zed Dart sources at recorded revisions.
- [x] Map source responsibilities and identify extension API questions.
- [x] Add application task and launch configuration examples.
- [x] Generate a sample Flutter counter application in `test/sample_app` with Zed configurations.
- [ ] Record the installed Zed, Dart extension, Flutter, and Dart versions.
- [ ] Use a small Flutter application to verify diagnostics, completion, formatting, launch, breakpoints, stepping, variable inspection, stop, and relaunch.
- [ ] Test Windows desktop or another available target with explicit device selection.
- [ ] Verify attach and FVM separately where supported by the available environment.

Acceptance: record observed results and reproducible failures. Source support alone does not count as a passing integration test.

## 2. Implement the first missing capability

Start with reliable SDK resolution and launch configuration handling if baseline testing confirms a gap. Prefer an upstream Dart extension contribution for shared behavior. If a separate Flutter debugger is justified, give it a distinct adapter identifier so both extensions can be installed together.

- [ ] Define explicit SDK override, project-local SDK or FVM, and PATH precedence.
- [ ] Validate request type and field types with actionable errors.
- [ ] Preserve supported SDK launch arguments and environment behavior.
- [ ] Respect a user-provided adapter path.
- [ ] Verify Windows `.bat` execution, paths containing spaces, and FVM invocation.
- [ ] Implement a debug scenario conversion or locator for common Flutter entry points.
- [ ] Add the extension manifest, adapter schema, and Rust crate only once the implementation route is selected from baseline evidence.

Acceptance: meaningful tests for configuration translation and SDK command construction, a successful WebAssembly build, and an actual launch and stop cycle in Zed. Do not duplicate Dart language-server registration as part of a debugger-only extension.

## 3. Prove hot reload integration

- [ ] Determine how the selected Zed release can send a custom request to an active Flutter DAP session.
- [ ] Verify `hotReload` preserves app state and `hotRestart` resets it.
- [ ] Handle no active session, startup, failures, and multiple sessions explicitly.
- [ ] Investigate save events and reload coalescing only after manual reload works.

Acceptance: reload changes a running application without creating another process. Restart behavior is distinguished from stopping and relaunching the debugger. If the public API cannot support the workflow, document the concrete host API gap before selecting a companion or upstream Zed change.

## 4. Improve the daily workflow

- [ ] Device discovery, selection, emulator launch, and disconnect handling.
- [ ] DevTools launch using the correct active session service URI.
- [ ] Test discovery, focused test execution, and structured results.
- [ ] Profile and release configurations where debugging capabilities differ.
- [ ] Multi-root projects, nested packages, and per-project SDK versions.

Acceptance: document supported platforms and targets, with integration evidence for each. Embedded inspector, previews, and editor decorations remain separate feasibility work.

## Validation rules

Keep configuration checks separate from end-to-end results. Run Rust checks and the WebAssembly build for Rust changes. When changing Flutter fixtures, run `flutter analyze && dart format .` from the fixture package outside the sandbox, then run tests appropriate to the change.

Do not analyze or format the entire `.reference/` tree. Run Flutter checks from `test/sample_app`, which contains the sample package. This repository currently has no Rust crate. Rust and Cargo were not found on the command PATH during the initial inspection, so extension compilation remains unverified.
