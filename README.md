# Flutter tooling for Zed

This project investigates how to bring Dart Code's Flutter development workflow to Zed. The initial research compares the VS Code implementation with Zed's existing Dart support and extension APIs.

The repository currently contains research, baseline configuration examples, and a [sample Flutter application](test/README.md). It does not yet contain an installable extension, and the configurations have not been exercised in Zed.

Start with [the source investigation](docs/upstream-investigation.md) and [the implementation roadmap](docs/roadmap.md).

The main finding is that [Dart-Code/Flutter](https://github.com/Dart-Code/Flutter) activates the Dart extension, while [Dart-Code/Dart-Code](https://github.com/Dart-Code/Dart-Code) implements the main Flutter workflow. [Zed's Dart extension](https://github.com/zed-extensions/dart) already includes a Flutter debug adapter launch path, so the first milestone is to measure that baseline and identify the missing integration.

## Repository contents

| Path | Purpose |
| --- | --- |
| `docs/upstream-investigation.md` | Pinned source map, feature mapping, and API constraints |
| `docs/roadmap.md` | Implementation stages and acceptance criteria |
| `examples/zed/` | Task and debug configurations for an existing Flutter application |
| `test/sample_app/` | Flutter counter app with Zed configurations for integration testing |
| `.reference/` | Ignored local upstream source checkouts |

## Try the baseline

Install the existing Dart extension in Zed and open a Flutter application's root folder. Merge [tasks.json](examples/zed/tasks.json) and [debug.json](examples/zed/debug.json) into that application's `.zed/` directory. Preserve any existing entries. Both Flutter and Dart must be visible to Zed's environment.

Use `task: spawn` for CLI tasks and `debugger: start` for the Flutter launch scenario. The example leaves device selection to Flutter. Add `"toolArgs": ["-d", "your-device-id"]` to select a device explicitly. The terminal run task supports Flutter's interactive `r` and `R` commands when supported by the target. Those keys do not establish hot reload support in Zed's debugger.

These are application configuration examples, not an installation of this project's future extension. A Rust toolchain and an actual Zed session will be needed for extension implementation and integration testing.
