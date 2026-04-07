# AGENTS.md - Commet Development Guidelines

## Overview
Commet is a Flutter-based Matrix client with a monorepo structure containing multiple packages (commet, tiamat, widgets). This file provides guidelines for agentic coding agents working in this repository.

---

## Build Commands

### Basic Workflow
```bash
# From repository root
cd commet

# Fetch dependencies
flutter pub get

# Run code generation (required before building)
dart run scripts/codegen.dart

# Run app (debug mode)
flutter run --dart-define BUILD_MODE=debug --dart-define PLATFORM=linux
```

### Build Flags
| Flag | Values | Description |
|------|--------|-------------|
| PLATFORM | desktop, mobile, linux, windows, macos, android, ios, web | Target platform |
| BUILD_MODE | release, debug | Build type |
| GIT_HASH | string | Git commit hash for info screen |
| VERSION_TAG | string | Version string for display |
| BUILD_DETAIL | string | Additional build metadata |

### Build Examples
```bash
# Linux release
flutter build linux --dart-define PLATFORM=linux --dart-define BUILD_MODE=release

# Android debug
flutter build apk --dart-define PLATFORM=android --dart-define BUILD_MODE=debug
```

### Code Generation
```bash
cd commet
dart run scripts/codegen.dart
```

---

## Linting & Analysis

### Run Analysis
```bash
cd commet
flutter analyze
```

### Analysis Config (analysis_options.yaml)
The project has minimal lint rules - mostly ignores curly braces in flow control and excludes generated files:
- Excludes: `*.g.dart`, `generated/*.dart`, `lib/main.widgetbook.dart`, `integration_test/**`

---

## Code Style Guidelines

### Import Organization
Organize imports in the following order (no blank lines between groups):
1. Dart core imports (`dart:async`, `dart:convert`, etc.)
2. External package imports (alphabetical)
3. Internal package imports (`package:commet/...`)
4. Relative imports

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:commet/client/client_manager.dart';
import 'package:commet/config/preferences.dart';
import '../ui/pages/main/main_page.dart';
```

### Disambiguation
When names conflict, use `as` aliases:
```dart
import 'package:commet/main.dart' as app_main;
import 'package:matrix/matrix.dart' as matrix;
import 'package:tiamat/tiamat.dart' as tiamat;
```

### Naming Conventions
- **Classes/Types**: PascalCase (`CallManager`, `MatrixClient`)
- **Methods/Variables**: camelCase (`currentUser`, `getMutedStateForClient`)
- **Constants**: PascalCase or camelCase with prefix (`static const String _voipMuteStateKey`)
- **Private members**: Prefix with underscore (`_lastKnownMutedState`, `_onMuteStateChanged`)
- **File names**: snake_case (`main_page_view_desktop.dart`, `call_manager.dart`)

### Type Usage
- Use explicit types for public APIs
- Use nullable types (`?`) for optional values
- Prefer `var` for local variables with clear initialization
- Use `late` for lazy initialization

```dart
// Good
Profile? current = state.currentUser;
var owningClient = clientManager!.clients.firstWhereOrNull((c) => c.self?.identifier == current!.identifier);
late String _id;

// Avoid
final current = state.currentUser;  // unclear type
```

### Error Handling
Use try-catch with the project's logging system:
```dart
try {
  await client.matrixClient.setAccountData(...);
} catch (e, s) {
  Log.onError(e, s);
}
```

### Null Safety
- Use null-aware operators (`?.`, `??`, `?.add()`)
- Use `!` only when absolutely certain value is non-null
- Prefer early returns for null checks

```dart
// Good
var data = client.matrixClient.accountData[_voipMuteStateKey];
if (data != null && data.content != null) {
  _lastKnownMutedState = data.content['muted'] as bool? ?? false;
}

// Avoid
var data = client.matrixClient.accountData[_voipMuteStateKey]!;  // may be null
```

### Stream Usage
- Always cancel subscriptions in `dispose()`
- Use `broadcast()` for streams that have multiple listeners
- Handle mounted check in stateful widgets

```dart
// In initState
sub = stream.listen((data) {
  if (mounted) setState(() {});
});

// In dispose
sub?.cancel();
```

---

## Project Structure

### Key Directories
```
commet/
├── lib/
│   ├── client/           # Client implementations (Matrix, etc.)
│   ├── config/           # Configuration (preferences, layout, build)
│   ├── ui/              # Flutter UI (pages, widgets, molecules)
│   ├── service/         # Background services
│   └── main.dart        # App entry point
├── integration_test/    # Integration tests
└── pubspec.yaml

tiamat/                   # UI component library
widgets/                  # Reusable widgets (calendar, matrix_widget_api)
```

### Component Pattern
Many features use a component pattern with:
- Abstract interface in `client/components/`
- Matrix implementation in `client/matrix/components/`
- Post-login initialization in `postLoginInit()`

---

## Testing

### Running Tests
```bash
# All tests
cd commet && flutter test

# Single test file
flutter test test/file_name_test.dart

# Single test
flutter test --name "test_name"
```

Note: The project has minimal test coverage currently.

---

## Platform-Specific Notes

### Linux Dependencies
```bash
sudo apt-get install -y ninja-build libgtk-3-dev libmpv-dev mpv ffmpeg libmimalloc-dev
```

### LiveKit Notes
The app uses LiveKit for VoIP calls. When modifying call-related code, ensure changes work for both LiveKit sessions and Matrix WebRTC sessions (1:1 calls).

---

## Common Patterns

### Matrix Account Data
Store user preferences using Matrix account data (synced across devices):
```dart
await client.matrixClient.setAccountData(userId, key, {"data": value});
var data = client.matrixClient.accountData[key];
```

### State Management
The app uses Provider for dependency injection and local state management with StatefulWidget and setState().

---

## Version
- Flutter SDK: >=3.6.0 <4.0.0
- Last updated: April 2026