# renovatebot/renovate#46801: Flutter SDK `dev.flutter.flutter-plugin-loader` Gradle plugin always fails lookup

Minimal reproduction for a Renovate "Suggest an idea" discussion (link below).

## Setup

`android/settings.gradle` is the unmodified Android `settings.gradle` that `flutter create` has generated since Flutter 3.16 (declarative plugin application, see [Flutter's migration guide](https://docs.flutter.dev/release/breaking-changes/flutter-gradle-plugin-apply)). `renovate.json` contains no settings: Renovate defaults only.

`dev.flutter.flutter-plugin-loader` is provided by the `includeBuild("$flutterSdkPath/packages/flutter_tools/gradle")` in `pluginManagement`, from the Flutter SDK on the developer's machine. It is not published to the Gradle Plugin Portal or any Maven repository, and its version moves with the Flutter SDK.

## Current behavior

The gradle manager extracts the plugin and looks it up, and the lookup can never succeed:

```
Failed to look up maven package dev.flutter.flutter-plugin-loader:dev.flutter.flutter-plugin-loader.gradle.plugin: no-result
```

Every Flutter app therefore gets a permanent lookup warning on its Dependency Dashboard. The real plugins in the same block (`com.android.application`, `org.jetbrains.kotlin.android`) resolve normally.

Reproduced with Renovate **44.140.0**:

```sh
npx renovate@44.140.0 --platform=local --dry-run=lookup --report-type=file --report-path=report.json
# report.json → android/settings.gradle:
#   dev.flutter.flutter-plugin-loader  skipReason: null  warnings: ["Failed to look up maven package ...: no-result"]
#   com.android.application            updates: 8.13.2, 9.4.1
#   org.jetbrains.kotlin.android       updates: 1.9.25, 2.4.20
```

## Expected behavior

The gradle manager skips the Flutter SDK's `dev.flutter.*` plugins at extraction, as the `pub` manager already does for Flutter SDK packages (`flutter_localizations`, `flutter_test`, …; renovatebot/renovate#24285). No lookup, and no warning.

## Link to the Renovate issue or Discussion

https://github.com/renovatebot/renovate/discussions/46801
