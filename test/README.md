# Flutter test application

`sample_app/` is a standard Flutter counter application for testing Dart and Flutter tooling in Zed. It includes Android, iOS, web, Windows, macOS, and Linux project files. Running a target requires its platform tooling and a supported host.

Open `test/sample_app` as the project folder in Zed so its `.zed/` configurations resolve paths correctly. Install Zed's existing Dart extension and make Flutter and Dart available in Zed's environment.

- Run `Flutter: run interactively` from `task: spawn` for the terminal workflow.
- Select `Flutter: launch application` from `debugger: start` for the debugger workflow.
- Use `Flutter: list devices` to find a target. Add `"toolArgs": ["-d", "your-device-id"]` to `.zed/debug.json` when explicit selection is needed.
- Place a breakpoint inside `_incrementCounter` in `lib/main.dart`, then press the app's increment button to check breakpoint and variable support.
- Change the title or theme while running to test hot reload. The counter provides visible state for checking whether reload preserves state.

From `test/sample_app`, validate changes with:

```powershell
flutter analyze && dart format .
flutter test
```

Analysis and widget tests check the application. Debugging and hot reload still require verification in Zed.
