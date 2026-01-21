# Auth State Management in Firebase UI Auth

## Overview

The `auth_state.dart` file defines the core state management system for Firebase UI Auth, providing a comprehensive set of authentication states and utilities for tracking and responding to authentication flow transitions. This file establishes the foundation for reactive authentication UI components.

## Core Architecture

### AuthState Abstract Class

The `AuthState` class serves as the base class for all authentication states in the Firebase UI Auth system.

```dart 49:64:packages/firebase_ui_auth/lib/src/auth_state.dart
abstract class AuthState {
  const AuthState();

  /// Returns current [AuthState] of the auth flow.
  /// Should be used only inside the widget that has an [AuthFlowBuilder] as
  /// an ancestor. Use [maybeOf] if there is a chance that the widget is used
  /// without [AuthFlowBuilder] as an ancestor.
  static AuthState of(BuildContext context) => maybeOf(context)!;

  /// Returns current [AuthState] of the auth flow.
  /// Could return null if no [AuthFlowBuilder] was found up  the widget tree.
  ///
  /// See [AuthFlowBuilder] for more examples.
  static AuthState? maybeOf(BuildContext context) =>
      context.dependOnInheritedWidgetOfExactType<AuthStateProvider>()?.state;
}
```

This abstract class provides static methods for accessing the current authentication state from the widget tree:

- `of(context)` - Returns the current auth state (throws if no provider found)
- `maybeOf(context)` - Returns the current auth state or null if no provider exists

## Authentication State Classes

### Basic Flow States

#### Uninitialized State

```dart 69:72:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.uninitialized}
/// A default [AuthState] for many auth flows.
/// {@endtemplate}
class Uninitialized extends AuthState {
  /// {@macro ffui.auth.auth_state.uninitialized}
  const Uninitialized();
}
```

The `Uninitialized` state represents the default starting state before any authentication process begins.

#### SigningIn State

```dart 80:83:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.signing_in}
/// Indicates that sign in is in progress.
/// Could be used to reflect the loading state on the ui.
///
/// See [AuthState] docs for usage examples.
/// {@endtemplate}
class SigningIn extends AuthState {
  /// {@macro ui.auth.auth_state.signing_in}
  const SigningIn();
}
```

The `SigningIn` state indicates that an authentication process is currently in progress, typically used to show loading indicators in the UI.

### Credential-Based States

#### CredentialReceived State

```dart 93:98:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.credential_received}
/// Indicates that the auth credential was successfully received.
/// This is an intermediate state that should transition to either [SignedIn],
/// [CredentialLinked] or [AuthFailed] depending on [AuthAction].
/// Could be used to reflect the loading state on the ui.
///
/// See [AuthState] docs for usage examples.
/// {@endtemplate}
class CredentialReceived extends AuthState {
  /// A credential that was received during auth flow.
  final AuthCredential credential;

  CredentialReceived(this.credential);
}
```

The `CredentialReceived` state represents an intermediate step where authentication credentials have been obtained but the authentication process is not yet complete. This state can transition to:

- `SignedIn` - Successful authentication
- `CredentialLinked` - Credential linked to existing account
- `AuthFailed` - Authentication failure

#### CredentialLinked State

```dart 106:115:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.credential_linked}
/// Indicates that the auth credential was successfully linked with the
/// currently signed in user account.
///
/// See [AuthState] docs for usage examples.
/// {@endtemplate}
class CredentialLinked extends AuthState {
  /// A credential that was linked with the currently signed in user account.
  final AuthCredential credential;

  /// An instance of the [User] the credential was associated with.
  final User user;

  /// {@macro ui.auth.auth_state.credential_linked}
  CredentialLinked(this.credential, this.user);
}
```

The `CredentialLinked` state indicates that a new authentication credential has been successfully linked to an existing user account, enabling multi-provider authentication.

### Error and Success States

#### AuthFailed State

```dart 123:134:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.auth_failed}
/// An [AuthState] that indicates that something went wrong during
/// authentication.
///
/// See [AuthState] docs for usage examples.
/// {@endtemplate}
class AuthFailed extends AuthState {
  /// The error that occurred during authentication.
  /// Often this is an instance of [FirebaseAuthException] that might contain
  /// more details about the error.
  ///
  /// There is an [ErrorText] widget that can be used to display error details
  /// in human readable form.
  final Exception exception;

  /// {@macro ui.auth.auth_state.auth_failed}
  AuthFailed(this.exception);
}
```

The `AuthFailed` state represents authentication failures, containing the exception that occurred. The Firebase UI Auth library provides an `ErrorText` widget for user-friendly error display.

#### SignedIn State

```dart 141:147:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.signed_in}
/// An [AuthState] that indicates that the user has successfully signed in.
///
/// See [AuthState] docs for usage examples.
/// {@endtemplate}
class SignedIn extends AuthState {
  /// An instance of the [User] that was signed in.
  final User? user;

  /// {@macro ui.auth.auth_state.signed_in}
  SignedIn(this.user);
}
```

The `SignedIn` state indicates successful authentication, containing the authenticated Firebase user object.

### Specialized States

#### UserCreated State

```dart 149:155:packages/firebase_ui_auth/lib/src/auth_state.dart
/// A state that indicates that a new user account was created.
class UserCreated extends AuthState {
  /// A [UserCredential] that was obtained during authentication process.
  final UserCredential credential;

  UserCreated(this.credential);
}
```

The `UserCreated` state is emitted when a new user account is successfully created during the authentication flow.

#### MFARequired State

```dart 160:165:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.mfa_required}
/// An [AuthState] that indicates that multi-factor authentication is required.
/// {@endtemplate}
class MFARequired extends AuthState {
  /// A multi-factor resolver that should be used to complete MFA.
  final MultiFactorResolver resolver;

  const MFARequired(this.resolver);
}
```

The `MFARequired` state indicates that multi-factor authentication is required to complete the sign-in process, providing the resolver needed to handle the MFA challenge.

## State Management Infrastructure

### AuthStateProvider

```dart 167:180:packages/firebase_ui_auth/lib/src/auth_state.dart
class AuthStateProvider extends InheritedWidget {
  final AuthState state;

  const AuthStateProvider({
    super.key,
    required super.child,
    required this.state,
  });

  @override
  bool updateShouldNotify(AuthStateProvider oldWidget) {
    return state != oldWidget.state;
  }
}
```

The `AuthStateProvider` is an `InheritedWidget` that propagates authentication state down the widget tree, enabling child widgets to access the current auth state via the static methods in `AuthState`.

## State Transition System

### AuthStateTransition

```dart 187:200:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.auth_state_transition}
/// A sub-type of the [Notification] that is used to notify about auth state
/// transitions. You could use [NotificationListener], but it is recommended
/// to use [AuthStateListener] instead.
/// {@endtemplate}
class AuthStateTransition<T extends AuthController> extends Notification {
  /// Previous [AuthState].
  final AuthState from;

  /// Current [AuthState].
  final AuthState to;

  /// An instance of [AuthController] that could be used to perform further
  /// actions of the auth flow.
  final T controller;

  /// {@macro ui.auth.auth_state.auth_state_transition}}
  AuthStateTransition(this.from, this.to, this.controller);
}
```

The `AuthStateTransition` notification is dispatched whenever the authentication state changes, containing:

- `from` - The previous authentication state
- `to` - The new authentication state
- `controller` - The auth controller instance for performing additional actions

### AuthStateListener Widget

```dart 229:257:packages/firebase_ui_auth/lib/src/auth_state.dart
/// {@template ui.auth.auth_state.auth_state_listener}
/// A [Widget] that could be used to listen auth state transitions.
///
/// For example, you could show a snackbar when some error occurs:
///
/// ```dart
/// AuthStateListener<EmailAuthController>(
///   child: LoginView(
///     actions: AuthAction.signIn,
///     providers: [EmailAuthProvider()],
///   ),
///   listener: (oldState, state, controller) {
///     if (state is AuthFailed) {
///       ScaffoldMessenger.of(context).showSnackBar(
///         SnackBar(content: ErrorText(exception: state.exception),
///       );
///     }
///   }
/// )
/// ```
/// {@endtemplate}
class AuthStateListener<T extends AuthController> extends StatelessWidget {
  final Widget child;
  final AuthStateListenerCallback<T> listener;

  const AuthStateListener({
    super.key,
    required this.child,
    required this.listener,
  });

  @override
  Widget build(BuildContext context) {
    return NotificationListener(
      onNotification: (notification) {
        if (notification is! AuthStateTransition<T>) {
          return false;
        }

        return listener(
              notification.from,
              notification.to,
              notification.controller,
            ) ??
            false;
      },
      child: child,
    );
  }
}
```

The `AuthStateListener` widget provides a convenient way to listen for authentication state transitions. It wraps a child widget and calls the provided listener callback whenever an `AuthStateTransition` notification is received.

## Usage Patterns

### State Access in Widgets

Widgets can access the current authentication state using the static methods:

```dart
// Using AuthState.of(context) - throws if no provider found
final state = AuthState.of(context);

// Using AuthState.maybeOf(context) - returns null if no provider found
final state = AuthState.maybeOf(context);
```

### State-Based UI Rendering

The authentication states enable conditional UI rendering:

```dart
class MyAuthWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final state = AuthState.of(context);

    if (state is Uninitialized) {
      return Text('Initializing...');
    } else if (state is SigningIn) {
      return CircularProgressIndicator();
    } else if (state is SignedIn) {
      return Text('Welcome, ${state.user?.displayName}');
    } else if (state is AuthFailed) {
      return Text('Error: ${state.exception}');
    }

    return Text('Unknown state');
  }
}
```

### Responding to State Transitions

The `AuthStateListener` can respond to state changes:

```dart
AuthStateListener<EmailAuthController>(
  listener: (oldState, newState, controller) {
    if (newState is AuthFailed) {
      // Show error message
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Authentication failed')),
      );
    } else if (newState is SignedIn) {
      // Navigate to home screen
      Navigator.of(context).pushReplacementNamed('/home');
    }
  },
  child: LoginForm(),
)
```

## Architecture Benefits

This state management system provides several key benefits:

1. **Type Safety**: Strongly typed state classes prevent invalid state transitions
2. **Reactive UI**: Widgets can automatically rebuild when authentication state changes
3. **Separation of Concerns**: UI logic is separated from authentication logic
4. **Composable**: States can be combined with other state management solutions
5. **Testable**: Each state is a simple data class that's easy to test

The design follows Flutter's reactive patterns while providing Firebase-specific authentication state management, making it easy to build responsive authentication UIs that react to authentication flow changes.
