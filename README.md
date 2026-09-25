# Heabron Mobile

A Flutter mobile application for the Heabron platform — serving farmers, cooperatives, and agribusiness stakeholders with wallet management, activity tracking, and more.

---

## Table of Contents

- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Modules](#modules)
- [Core Utilities](#core-utilities)
- [API & Networking](#api--networking)
  - [ServerResponse Model](#serverresponse-model)
  - [Pagination — ResponseMeta](#pagination--responsemeta)
  - [Error Handling — ResponseError](#error-handling--responseerror)
- [Environment Configuration](#environment-configuration)
- [App Entry Points](#app-entry-points)
- [Notes](#notes)
- [Resources](#resources)

---

## Getting Started

### Prerequisites

- Flutter SDK `>=3.0.0`
- Dart SDK `>=3.0.0`
- Android Studio / Xcode (for device emulation)
  > **Note:** This app currently targets Android and iOS platforms exclusively. Web and Windows support has been removed.

### Setup

```bash
# Clone the repository
git clone <repo-url>
cd heabron_mobile

# Install dependencies
flutter pub get

# Configure environment variables
copy env.example.json env.json
# Fill in the required values in env.json

# Run the app
flutter run --dart-define-from-file=env.json
```

> The project uses `env.json` together with Dart defines, and `.vscode/launch.json` is already configured to load it.

---

## Project Structure

```
lib/
└── src/
    ├── core/               # App-wide utilities, theme, routing
    │   ├── route/          # Named route definitions
    │   └── utils/
    │       ├── base/       # Base DTOs and repository classes
    │       ├── exception/  # Custom exception types
    │       ├── extensions/ # Dart extension methods
    │       ├── listeners/  # App-level listeners
    │       ├── mixins/     # Shared mixins
    │       ├── api_constants.dart
    │       ├── app_assets.dart
    │       ├── app_colors.dart
    │       ├── app_environmental_variables.dart
    │       ├── app_theme.dart
    │       ├── common_libs.dart
    │       └── typedefs.dart
    ├── modules/            # Feature modules (clean architecture)
    │   ├── activities/
    │   ├── authentication/
    │   ├── cooperatives/
    │   ├── dashboard/
    │   ├── farmers/
    │   ├── home/
    │   ├── notifications/
    │   ├── profile/
    │   ├── startup/
    │   └── wallet/
    ├── services/           # Third-party and platform services
    └── shared/             # Shared widgets and components
```

Each feature module follows **Clean Architecture**:

```
<module>/
├── data/
│   ├── model/              # API response models
│   └── repository/         # Remote repository implementations
├── domain/
│   ├── dto/                # Domain-specific DTO classes
│   ├── entity/             # Domain entities
│   └── repository/         # Abstract repository contracts
└── presentation/
    ├── controller/         # State controllers and providers
    ├── view/               # Page/Screen widgets
    └── widgets/            # Reusable UI components
```

---

## Modules

| Module | Description |
|---|---|
| `authentication` | Login, registration, identity verification, password reset |
| `dashboard` | Main dashboard overview with quick insights |
| `home` | Home screen, navigation shell |
| `wallet` | Wallet overview, transfers, settlement requests, transaction history |
| `activities` | Activity feed, tabs for recent and filtered actions |
| `farmers` | Farmer profiles, field notes, production yield tracking (with edit support), market access |
| `cooperatives` | Cooperative details, farmer onboarding, financing requests |
| `notifications` | Notifications listing and alerts |
| `profile` | Profile settings, bank account settings, review flows |
| `startup` | Splash screen, pre-auth flow, onboarding scaffolding |

---

## Core Utilities

| File | Purpose |
|---|---|
| `api_constants.dart` | Environment-based API base URLs and endpoint constants |
| `app_colors.dart` | Color tokens for the design system |
| `app_theme.dart` | App theme definitions and text styling |
| `app_assets.dart` | Asset paths for images, icons, and fonts |
| `common_libs.dart` | Shared imports and helper aliases |
| `typedefs.dart` | Global type aliases like `JSON` |
| `app_environmental_variables.dart` | Dart `String.fromEnvironment` accessors for env keys |
| `generate_route.dart` | Centralized named route generation with `ThemeScaffold` wrapper |

---

## API & Networking

### ServerResponse Model

**File:** `lib/src/core/utils/base/dto/server_response.dart`

All API responses are parsed into a `ServerResponse` object.

**Class:** `ServerResponse`

| Property | Type | Description |
|---|---|---|
| `success` | `bool?` | Whether the request succeeded |
| `data` | `dynamic` | Response payload (`Map` or `List`) |
| `meta` | `ResponseMeta?` | Pagination metadata |
| `error` | `ResponseError?` | Structured error object |

**Getters:**

| Getter | Type | Description |
|---|---|---|
| `isSuccess` | `bool` | Null-safe alias for `success ?? false` |
| `errorMessage` | `String` | Shortcut for `error?.message ?? ''` |

**Usage example:**

```dart
final response = ServerResponse.fromJson(jsonMap);
if (response.isSuccess) {
  final items = response.data as List;
} else {
  print(response.errorMessage);
}
```

---

### Pagination — ResponseMeta

| Property | Type | Description |
|---|---|---|
| `page` | `int` | Current page number |
| `pageSize` | `int` | Number of items per page |
| `total` | `int` | Total record count |
| `totalPages` | `int` | Total pages available |
| `hasNextPage` | `bool` | `true` when more pages remain |

**Usage example:**

```dart
final meta = response.meta;
if (meta != null && meta.hasNextPage) {
  loadNextPage(meta.page + 1);
}
```

---

### Error Handling — ResponseError

| Property | Type | Description |
|---|---|---|
| `code` | `String` | Machine-readable error code |
| `message` | `String` | Human-readable error text |
| `details` | `List<ResponseErrorDetail>` | Validation or field errors |

**Usage example:**

```dart
if (!response.isSuccess) {
  final error = response.error;
  print(error?.code);
  print(error?.message);
  for (final detail in error?.details ?? []) {
    print('${detail.field}: ${detail.message}');
  }
}
```

---

## Environment Configuration

Environment values are loaded at runtime via Dart defines from `env.json`.

**Example:** `env_example.json`

```json
{
  "STAGING_BASE_URL": "https://heabron-coopscore.vercel.app/api",
  "PRODUCTION_BASE_URL": "https://heabron-coopscore.vercel.app/api"
}
```

**Current launch configuration:** `.vscode/launch.json`

```json
"args": [
  "--dart-define-from-file=env.json"
]
```

**Access values in Dart:**

```dart
final stagingUrl = AppEnvironmentalVariables.stagingbaseUrl;
final productionUrl = AppEnvironmentalVariables.productionbaseUrl;
```

**Run commands:**

```bash
flutter pub get
flutter run --dart-define-from-file=env.json
```

---

## App Entry Points

| File | Purpose |
|---|---|
| `lib/main.dart` | App bootstrap; Hive initialization; portrait orientation lock |
| `lib/app.dart` | `MaterialApp` setup, theme, route generation, toast wrapper |
| `lib/src/core/route/generate_route.dart` | Maps named routes to screens with `ThemeScaffold` wrappers |
| `lib/src/core/utils/app_environmental_variables.dart` | Environment string constants from Dart defines |

**Initial route:** `SplashView.route`

**State management root:** `ProviderScope` using `hooks_riverpod`

---

## Notes

- Uses `Hive` for local caching.
- Uses `flutter_screenutil` for responsive layouts.
- Uses `toastification` for app toast notifications.
- Uses `RouteGenerator.generateRoute` for centralized named navigation.

---

## Resources

- [Flutter Documentation](https://docs.flutter.dev/)
- [Riverpod](https://riverpod.dev/)
- [Dart Language](https://dart.dev/guides)
- [Toastification package](https://pub.dev/packages/toastification)
