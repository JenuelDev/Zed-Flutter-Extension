# Upstream investigation

Inspected on 2026-11-09. Findings below describe the checked-out revisions. Moving documentation links should be rechecked during implementation.

## Reference revisions

| Repository | Inspected commit | Local checkout |
| --- | --- | --- |
| Dart-Code/Flutter | `d648300edf0a145041c2739f943db0a4cfd145db` | `.reference/flutter` |
| Dart-Code/Dart-Code | `82e9d1371e89b966e0861fdfdeee1a3ca3c9a921` | `.reference/dart-code` |
| zed-extensions/dart | `fba74dfa66de4d123206dff56bf0b1fa20e3505d` | `.reference/zed-dart` |

The checkouts are research inputs and are excluded from this repository's tracked files. Clone each repository at the revision above to reproduce the source inspection.

## Where the behavior lives

The Flutter extension's [manifest](https://github.com/Dart-Code/Flutter/blob/d648300edf0a145041c2739f943db0a4cfd145db/package.json) depends on `Dart-Code.dart-code`. Its [activation function](https://github.com/Dart-Code/Flutter/blob/d648300edf0a145041c2739f943db0a4cfd145db/src/extension.ts) activates that dependency and constructs `SdkCommands`. The command registration inside `src/commands/sdk.ts` is commented out at this revision. Porting this repository alone would therefore omit the main Flutter tooling.

Use the following paths inside `.reference/dart-code` for further investigation:

| Responsibility | Source entry points |
| --- | --- |
| Activation and service wiring | `src/extension/extension.ts` |
| Workspace and SDK selection | `src/shared/workspace.ts`, `src/shared/utils/workspace.ts`, `src/shared/vscode/sdk_manager.ts` |
| Launch configuration resolution | `src/extension/providers/debug_config_provider.ts` |
| SDK debug adapter startup | `src/extension/providers/debug_adapter_descriptor_factory.ts` |
| Reload and restart commands | `src/extension/commands/debug.ts` |
| Reload on save | `src/extension/dart/hot_reload_save_handler.ts` |
| Flutter daemon and protocol | `src/extension/flutter/flutter_daemon.ts`, `src/shared/flutter/daemon_interfaces.ts` |
| Device and emulator workflow | `src/shared/vscode/device_manager.ts` |
| CLI tasks and Flutter commands | `src/extension/flutter/flutter_task_provider.ts`, `src/extension/commands/flutter.ts` |
| DevTools lifecycle and embedded UI | `src/extension/sdk/dev_tools/manager.ts`, `src/extension/sdk/dev_tools/embedded_view.ts` |
| Tests and discovery | `src/extension/test/vs_test_controller.ts`, `src/extension/lsp/test_discoverer.ts` |
| Editor-specific widget decorations | `src/extension/decorations/flutter_ui_guides_decorations.ts` |

The [adapter factory](https://github.com/Dart-Code/Dart-Code/blob/82e9d1371e89b966e0861fdfdeee1a3ca3c9a921/src/extension/providers/debug_adapter_descriptor_factory.ts) launches SDK-provided adapters with `debug_adapter`, including `--test` for test adapters. This lets the project reuse the SDK's debugging implementation instead of translating the legacy TypeScript debugger.

The [debug commands](https://github.com/Dart-Code/Dart-Code/blob/82e9d1371e89b966e0861fdfdeee1a3ca3c9a921/src/extension/commands/debug.ts) send custom DAP requests named `hotReload` and `hotRestart` to active sessions. Reload on save also requires document lifecycle events and session state. Merely launching a DAP server does not supply this editor integration.

## Existing Zed baseline

The inspected [Zed Dart implementation](https://github.com/zed-extensions/dart/blob/fba74dfa66de4d123206dff56bf0b1fa20e3505d/src/dart.rs) already provides:

- Dart LSP startup and workspace settings forwarding.
- Dart and Flutter SDK debug adapter startup through an adapter named `Dart`.
- Launch or attach request selection, program arguments, working directory, environment, and VM service URI forwarding.
- Flutter device selection through `toolArgs`, with explicit `deviceId` translated to `-d` when no device argument is detected.
- An optional FVM launch path.

These are observations from source, not results of running an application in Zed. The inspected extension manifest reports version `0.4.1` and its Cargo manifest uses `zed_extension_api = "0.7.0"`.

Potential improvement candidates visible in the source include strict type validation, honoring the user-provided adapter path, preserving supported launch fields instead of rebuilding a limited subset, and task-to-debug scenario conversion. These need targeted reproduction before being classified as defects. The Windows FVM branch passes the platform tool name, such as `flutter.bat`, to FVM, which also needs verification.

## Mapping to Zed

| Flutter workflow | Proposed route | Status or constraint |
| --- | --- | --- |
| Completion, diagnostics, navigation, formatting | Existing Dart extension and SDK LSP | Source baseline exists; verify in Zed |
| Launch, attach, breakpoints, stepping, variables | SDK DAP through Zed debugger | Source baseline exists; verify full session |
| Run, analyze, format, packages, tests | Zed terminal tasks | Configuration examples included |
| SDK discovery and FVM | Worktree environment plus explicit configuration | Define precedence and test Windows paths |
| Automatic launch configurations | DAP scenario conversion and debug locators | Supported API hooks; Flutter mapping needs implementation |
| Device list and emulator launch | CLI tasks initially; daemon integration later | Interactive picker needs a proven UI route |
| Hot reload and hot restart | Custom DAP requests or another proven SDK control route | Active-session command API not found in inspected extension trait |
| Hot reload on save | Editor save events plus active-session routing | Requires feasibility work before a parity promise |
| DevTools and widget inspector | External DevTools first | Session URI discovery and browser opening need verification |
| Embedded DevTools, widget preview, UI guides | Zed-specific integration | No equivalent arbitrary panel or decoration API found in inspected trait |
| Test explorer | CLI tests first, structured integration later | VS Code Testing API code cannot transfer directly |

[Zed extension development](https://zed.dev/docs/extensions/developing-extensions) uses Rust compiled to WebAssembly. The documented build target is `wasm32-wasip2`. [Debugger extensions](https://zed.dev/docs/extensions/debugger-extensions) register adapters in `extension.toml`, supply a configuration schema, and implement `get_dap_binary` and `dap_request_kind`. Scenario conversion and debug locators support automatic configurations.

The inspected [Extension trait](https://github.com/zed-industries/zed/blob/main/crates/extension_api/src/extension_api.rs) exposes adapter creation and configuration hooks, but no general active-session custom request, save-event subscription, or arbitrary editor panel registration. This is a feasibility inference from the public surface inspected, not proof that every alternative integration is impossible. Check the selected Zed release and host code before deciding to use a companion process or request a Zed API change.

## Proposed architecture

Keep Dart syntax and language intelligence in the existing Dart extension. Prefer improvements there for shared SDK or DAP behavior. Use this repository to develop and validate Flutter-specific additions, with a standalone debugger extension only if it adds a distinct capability and can coexist with Dart support.

The first implementation should be a small Rust adapter or locator layer around SDK tooling. Separate SDK resolution, configuration validation, and platform command construction so they can be tested without Zed. Do not introduce a persistent daemon until a concrete feature requires one and its lifecycle can be implemented through supported APIs.

The Flutter daemon serves device and emulator management; the SDK debug adapter serves debugging and app lifecycle requests. Keep those protocol roles separate. If a companion process becomes necessary, specify transport, startup, shutdown, session ownership, and failure recovery before building it.

## Reuse and attribution

The inspected Flutter and Dart Code repositories contain MIT licenses. The Zed Dart repository contains an Apache-2.0 license. This research has not copied their implementation into a new extension. Preserve the applicable license and attribution material when adapting code, snippets, schemas, or grammar queries. Select this project's license before distributing implementation derived from upstream.
