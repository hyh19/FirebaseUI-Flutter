# AuthStateChangeAction Class

## Overview

`AuthStateChangeAction` is a generic action class in FirebaseUI Auth that enables developers to listen for specific authentication state transitions and execute custom callbacks when those states are reached. It's part of the FirebaseUI Actions system that provides a declarative way to handle authentication flow events.

## Generic Type Parameter

The class uses a generic type parameter `T extends AuthState`, which allows you to specify exactly which authentication state you want to listen for. This provides type safety and ensures that your callback receives the correct state type.

```dart 53:53:packages/firebase_ui_auth/lib/src/actions.dart
class AuthStateChangeAction<T extends AuthState> extends FirebaseUIAction {
```

## Constructor and Properties

### Callback Property

The core property is a callback function that accepts a `BuildContext` and the specific auth state of type `T`:

```dart 54:56:packages/firebase_ui_auth/lib/src/actions.dart
  /// A callback that is being called when underlying auth flow transitioned to
  /// a state of type [T].
  final void Function(BuildContext context, T state) callback;
```

### Constructor

The constructor simply takes the callback function:

```dart 58:59:packages/firebase_ui_auth/lib/src/actions.dart
  /// {@macro ui.auth.actions.auth_state_change_action}
  AuthStateChangeAction(this.callback);
```

## Core Methods

### matches() Method

This method determines whether a given authentication state matches the expected type `T`:

```dart 61:62:packages/firebase_ui_auth/lib/src/actions.dart
  /// Verifies that a current [state] is a [T]
  bool matches(AuthState state) => state is T;
```

It uses Dart's `is` operator to perform a runtime type check, ensuring type safety at runtime.

### invoke() Method

This method executes the callback with the provided context and state:

```dart 64:65:packages/firebase_ui_auth/lib/src/actions.dart
  /// Invokes the callback with the provided [context] and [state].
  void invoke(BuildContext context, T state) => callback(context, state);
```

## Usage Pattern

`AuthStateChangeAction` is typically used within authentication screens like `SignInScreen` to perform actions when users transition to specific authentication states. Here's the canonical usage pattern:

```dart 44:50:packages/firebase_ui_auth/lib/src/actions.dart
/// SignInScreen(
///   actions: [
///     AuthStateChangeAction<SignedIn>((context, state) {
///       Navigator.pushReplacementNamed(context, '/home');
///     }),
///   ],
/// );
```

In this example:

- `<SignedIn>` specifies that we want to listen for the `SignedIn` state
- The callback receives the `SignedIn` state object, which contains user information
- Common actions include navigation, showing dialogs, or updating app state

## Integration with FirebaseUI Actions System

`AuthStateChangeAction` extends `FirebaseUIAction` and integrates with the broader FirebaseUI Actions system:

1. **FirebaseUIActions Widget**: Provides actions down the widget tree via an inherited widget pattern
2. **Auth State Listener**: The `_FlutterfireUIAuthActionsElement` listens for authentication state changes
3. **Action Execution**: When a state change occurs, it iterates through all actions and calls `matches()` and `invoke()` for matching actions

```dart 181:186:packages/firebase_ui_auth/lib/src/actions.dart
        for (final action in widget.actions) {
          if (action is AuthStateChangeAction && action.matches(newState)) {
            _controllerRegistry[newState] = controller;
            action.invoke(this, newState);
            _controllerRegistry.remove(newState);
          }
        }
```

## Type Safety Benefits

The generic type parameter provides several benefits:

1. **Compile-time Safety**: Ensures you can only listen for valid `AuthState` subtypes
2. **Callback Type Safety**: The callback parameter is strongly typed to the specific state type
3. **IntelliSense Support**: IDEs can provide better autocomplete and error detection

## Common Auth States

While `AuthState` is the base class, common states you might listen for include:

- `SignedIn` - User successfully signed in
- `SigningIn` - User is in the process of signing in
- `SigningOut` - User is signing out
- Various error states and intermediate states

## Controller Access

When an action is invoked, you can access the `AuthController` that triggered the state change using the `getControllerForState()` function:

```dart 218:229:packages/firebase_ui_auth/lib/src/actions.dart
AuthController getControllerForState(AuthState state) {
  final ctrl = _controllerRegistry[state];

  if (ctrl == null) {
    throw StateError(
      'Quering controller for an auth state is only allowed '
      'from FirebaseUIAction callback',
    );
  }

  return ctrl;
}
```

This allows you to inspect which authentication method was used (email, phone, OAuth provider, etc.).

## Example: Advanced Usage

```dart
SignInScreen(
  actions: [
    AuthStateChangeAction<SignedIn>((context, state) {
      // Navigate to home
      Navigator.pushReplacementNamed(context, '/home');
      
      // Log analytics event
      Analytics.logEvent('user_signed_in');
      
      // Access the controller to determine auth method
      final controller = getControllerForState(state);
      if (controller is EmailAuthController) {
        Analytics.logEvent('signed_in_with_email');
      } else if (controller is GoogleAuthController) {
        Analytics.logEvent('signed_in_with_google');
      }
    }),
  ],
);
```

## Best Practices

1. **Keep Callbacks Simple**: Actions should be lightweight; complex logic belongs in your app's business logic layer
2. **Use Specific State Types**: Prefer specific states like `SignedIn` over generic `AuthState` for better type safety
3. **Handle Navigation Carefully**: Use appropriate navigation methods (`pushReplacement`, `pushAndRemoveUntil`, etc.) based on your app's flow
4. **Consider Error States**: Listen for error states to provide user feedback
5. **Test Actions**: Ensure your actions work correctly in different authentication scenarios

## Threading and Lifecycle

Actions are executed synchronously as part of the Flutter build process. The `_controllerRegistry` ensures that controller access is only available during action execution, preventing memory leaks and ensuring proper lifecycle management.
