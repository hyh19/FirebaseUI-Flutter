# FirebaseUIActions and _FlutterfireUIAuthActionsElement

## Overview

The `FirebaseUIActions` class is an inherited widget that provides a mechanism for handling authentication state changes through a list of configurable actions. It works in conjunction with the `_FlutterfireUIAuthActionsElement` to listen for authentication state changes and invoke appropriate actions.

## FirebaseUIActions Class

`FirebaseUIActions` is an inherited widget that propagates a list of `FirebaseUIAction` objects down the widget tree, allowing descendant widgets to access and respond to authentication-related actions.

### Key Properties

- `actions`: A list of `FirebaseUIAction` objects that define the available actions

### Static Methods

#### maybeOf(BuildContext context)

```dart 125:127:packages/firebase_ui_auth/lib/src/actions.dart
static FirebaseUIActions? maybeOf(BuildContext context) {
  return context.dependOnInheritedWidgetOfExactType<FirebaseUIActions>();
}
```

Looks up an instance of `FirebaseUIActions` in the widget tree. Returns `null` if no instance is found.

#### inherit()

```dart 131:151:packages/firebase_ui_auth/lib/src/actions.dart
static Widget inherit({
  /// A [BuildContext] to inherit from.
  required BuildContext from,

  /// A [Widget] to wrap with [FirebaseUIActions].
  required Widget child,

  /// A list of [FirebaseUIAction]s to provide to the [child].
  List<FirebaseUIAction> actions = const [],
}) {
  final w = maybeOf(from);

  if (w != null) {
    return FirebaseUIActions(
      actions: [...w.actions, ...actions],
      child: child,
    );
  }

  return child;
}
```

This method allows inheriting existing actions from the context and optionally adding new ones. It creates a new `FirebaseUIActions` widget with the combined list of actions from the parent and the newly provided actions.

### Widget Lifecycle

```dart 160:163:packages/firebase_ui_auth/lib/src/actions.dart
@override
bool updateShouldNotify(FirebaseUIActions oldWidget) {
  return oldWidget.actions != actions;
}
```

The widget only notifies descendants when the actions list has actually changed.

```dart 165:168:packages/firebase_ui_auth/lib/src/actions.dart
@override
InheritedElement createElement() {
  return _FlutterfireUIAuthActionsElement(this);
}
```

Instead of using the standard `InheritedElement`, it creates a custom `_FlutterfireUIAuthActionsElement` that provides additional functionality.

## _FlutterfireUIAuthActionsElement Class

This is a custom `InheritedElement` that wraps the child widget with an `AuthStateListener` to handle authentication state changes.

### Key Functionality

```dart 178:193:packages/firebase_ui_auth/lib/src/actions.dart
@override
Widget build() {
  return AuthStateListener<AuthController>(
    listener: (oldState, newState, controller) {
      for (final action in widget.actions) {
        if (action is AuthStateChangeAction && action.matches(newState)) {
          _controllerRegistry[newState] = controller;
          action.invoke(this, newState);
          _controllerRegistry.remove(newState);
        }
      }

      return null;
    },
    child: super.build(),
  );
}
```

The core functionality revolves around the `AuthStateListener`:

1. **State Listening**: Listens for changes in authentication state
2. **Action Filtering**: Iterates through all available actions
3. **Action Invocation**: For actions that are `AuthStateChangeAction` instances and match the new state, it:
   - Registers the controller in a global registry (`_controllerRegistry`)
   - Invokes the action with the current element and new state
   - Cleans up the controller from the registry after invocation

### Controller Registry

The `_controllerRegistry` appears to be a global mechanism for temporarily storing authentication controllers during action execution. This allows actions to access the current controller without having to pass it explicitly.

## Usage Pattern

This system enables a declarative approach to handling authentication state changes:

1. Define `AuthStateChangeAction` objects that specify which auth states they respond to
2. Wrap UI components with `FirebaseUIActions` to provide these actions
3. When auth state changes occur, matching actions are automatically invoked

## Design Benefits

- **Separation of Concerns**: Actions are decoupled from the UI components
- **Composability**: Actions can be inherited and combined from parent contexts
- **Automatic Cleanup**: Controllers are automatically registered and cleaned up
- **Type Safety**: Uses Dart's type system to ensure only compatible actions are invoked

This pattern provides a clean, reactive way to handle authentication state transitions in Flutter applications using Firebase UI.
