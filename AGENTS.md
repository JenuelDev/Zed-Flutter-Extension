# Project instructions

## Flutter and Dart commands

- Run all `flutter` and `dart` commands outside the sandbox. SDK startup reads Git metadata and writes SDK caches outside this workspace.
- Request tool approval when needed and prefer persistent `flutter` and `dart` prefix approvals. Execute routine commands directly instead of asking the user to run them.
- When working on Flutter code, run `flutter analyze && dart format .` from the relevant Flutter package. Do not run these commands against the upstream research checkouts or this documentation-only root.

## Writing

- Never use the em dash character (Unicode U+2014) in responses or generated text. Use commas, parentheses, colons, semicolons, or separate sentences.

## Source research

- Keep upstream checkouts inside `.reference/`, which is ignored by Git.
- Record inspected revisions and link findings to source files.
- Distinguish observed behavior, proposed behavior, and behavior verified in Zed.
- Preserve upstream license notices whenever copying or adapting source.
