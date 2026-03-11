# MyNotes

A cross-platform Flutter notes application with Firebase authentication and local SQLite persistence.

## Features

- **Authentication** — Register, log in, log out, email verification, and password reset via Firebase Auth
- **Notes** — Create, read, update, and delete personal notes
- **Offline support** — Local caching with SQLite via `sqflite`
- **Share** — Share notes to other apps using `share_plus`
- **Localization** — English and Swedish (`intl` / `flutter_localizations`)
- **State management** — BLoC pattern with `flutter_bloc`

## Tech Stack

| Layer        | Technology                       |
| ------------ | -------------------------------- |
| UI           | Flutter (Material 3)             |
| State        | BLoC / Cubit (`flutter_bloc`)    |
| Auth         | Firebase Authentication          |
| Remote DB    | Cloud Firestore                  |
| Local DB     | SQLite (`sqflite`)               |
| Localization | `flutter_localizations` + `intl` |

## Getting Started

### Prerequisites

- Flutter SDK `^3.10.0` (beta channel)
- A Firebase project with **Authentication** and **Firestore** enabled
- `flutterfire_cli` configured (or replace `lib/firebase_options.dart` with your own)

### Setup

```bash
# 1. Clone the repository
git clone <repo-url>
cd mynotes

# 2. Install dependencies
flutter pub get

# 3. Run the app
flutter run
```

> Ensure `google-services.json` (Android) and `GoogleService-Info.plist` (iOS) are placed in the correct platform directories before building.

## Project Structure

```
lib/
├── constants/        # Route name constants
├── enums/            # Shared enums (e.g., menu actions)
├── extensions/       # BuildContext & List extension methods
├── helpers/          # UI helpers (loading screen)
├── l10n/             # ARB localization files (en, sv)
├── services/
│   ├── auth/         # Auth service, provider, BLoC, and exceptions
│   └── crud/         # Note CRUD service and exceptions
├── utilities/        # Generic dialogs and utilities
├── views/            # All UI screens (login, register, notes, etc.)
└── main.dart
```

## State Management — BLoC

This app uses the [BLoC (Business Logic Component)](https://bloclibrary.dev) pattern to separate UI from business logic.

### How it works

```
UI  ──(Event)──▶  BLoC  ──(State)──▶  UI
```

1. **Events** — immutable actions dispatched from the UI (e.g. a button tap).
2. **BLoC** — receives events, executes business logic (auth calls, DB queries), and emits new states.
3. **States** — immutable snapshots of the app that the UI rebuilds from.

### Auth BLoC

Located in `lib/services/auth/bloc/`.

#### Events (`auth_event.dart`)

| Event                            | Trigger                                    |
| -------------------------------- | ------------------------------------------ |
| `AuthEventInitialize`            | App startup — checks existing auth session |
| `AuthEventLogIn`                 | User submits login form                    |
| `AuthEventRegister`              | User submits registration form             |
| `AuthEventLogOut`                | User taps log out                          |
| `AuthEventSendEmailVerification` | User requests verification email           |
| `AuthEventForgotPassword`        | User submits forgot-password form          |
| `AuthEventShouldRegister`        | Navigate to registration screen            |

#### States (`auth_state.dart`)

| State                        | Description                                  |
| ---------------------------- | -------------------------------------------- |
| `AuthStateUninitialized`     | App is starting, session not yet checked     |
| `AuthStateLoggedOut`         | No active session; shows login screen        |
| `AuthStateLoggedIn`          | Authenticated; carries the `AuthUser` object |
| `AuthStateNeedsVerification` | Logged in but email not yet verified         |
| `AuthStateRegistering`       | Registration flow is active                  |
| `AuthStateForgotPassword`    | Password reset flow is active                |

All states carry `isLoading` and an optional `loadingText` field so the UI can show a global loading overlay without duplicating that logic in every screen.

### Widget integration

`BlocProvider` at the root of the widget tree creates the single `AuthBloc` instance. Screens consume it with `BlocConsumer` — `listener` handles side-effects (showing/hiding the loading screen) while `builder` rebuilds the UI tree based on the current state:

```dart
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state.isLoading) {
      LoadingScreen().show(context: context, text: state.loadingText!);
    } else {
      LoadingScreen().hide();
    }
  },
  builder: (context, state) {
    if (state is AuthStateLoggedIn) return const NotesView();
    if (state is AuthStateLoggedOut) return const LoginView();
    // ...
  },
)
```

---

## Running Tests

```bash
flutter test
```
