# AuthFlowBuilder Widget Explanation

## Overview

The `AuthFlowBuilder` is a generic Flutter widget that provides a flexible way to build custom authentication UI flows in Firebase UI Auth. It acts as a bridge between authentication logic (AuthFlows) and the widget tree, allowing developers to create custom authentication interfaces while leveraging the built-in authentication functionality.

## Key Components

### Type Definitions

#### AuthFlowBuilderCallback<T extends AuthController\>

```dart 17:29:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
typedef AuthFlowBuilderCallback<T extends AuthController> = Widget Function(
  BuildContext context,

  /// Current [AuthState] of the [AuthFlow].
  AuthState state,

  /// An instance of [AuthController] that could be used to control the
  /// [AuthFlow].
  T ctrl,

  /// A [Widget] that was provided to the [AuthFlowBuilder].
  Widget? child,
);
```

This callback is invoked whenever the AuthFlow's state changes, allowing developers to build custom UI based on the current authentication state. It receives:

- `context`: The build context
- `state`: Current authentication state (e.g., AwaitingEmailAndPassword, SigningIn, AuthFailed)
- `ctrl`: The auth controller for manipulating the flow
- `child`: An optional pre-built child widget

#### StateTransitionListener<T extends AuthController\>

```dart 36:46:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
typedef StateTransitionListener<T extends AuthController> = void Function(
  /// Previous state of the [AuthFlow].
  AuthState oldState,

  /// Current state of the [AuthFlow].
  AuthState newState,

  /// An instance of the [AuthController] that could be used to manipulate the
  /// [AuthFlow].
  T controller,
);
```

This callback is invoked when the auth flow transitions between states, called before the widget rebuilds. It's useful for side effects like navigation or logging.

## Main Class: AuthFlowBuilder<T extends AuthController\>

### Static Members

#### Flow Registry

```dart 104:118:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
static final _flows = <Object, AuthFlow>{};

/// Resolves an [AuthController] by the [flowKey].
static T? getController<T extends AuthController>(Object flowKey) {
  final flow = _flows[flowKey];
  if (flow == null) return null;
  return flow as T;
}

/// Returns a current [AuthState] of the [AuthFlow] given the [flowKey].
static AuthState? getState(Object flowKey) {
  final flow = _flows[flowKey];
  if (flow == null) return null;
  return flow.value;
}
```

The widget maintains a static registry of AuthFlow instances keyed by flowKey. This allows accessing controllers and states globally across the widget tree.

### Constructor Parameters

```dart 159:173:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
const AuthFlowBuilder({
  super.key,
  this.flowKey,
  this.action,
  this.builder,
  this.onComplete,
  this.child,
  this.listener,
  this.provider,
  this.auth,
  this.flow,
}) : assert(
        builder != null || child != null,
        'Either child or builder should be provided',
      );
```

- `flowKey`: Unique identifier for the auth flow (optional)
- `action`: Authentication action (signIn, signUp, link)
- `builder`: Custom builder function for the UI
- `onComplete`: Callback when auth flow completes
- `child`: Pre-built child widget
- `listener`: State transition listener
- `provider`: Auth provider instance
- `auth`: FirebaseAuth instance
- `flow`: Pre-configured AuthFlow instance

### Flow Creation Logic

The widget automatically creates appropriate AuthFlow instances based on the provider type:

```dart 233:269:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
AuthFlow createFlow() {
  if (widget.flowKey != null) {
    final existingFlow = AuthFlowBuilder._flows[widget.flowKey!];
    if (existingFlow != null) {
      return existingFlow;
    }
  }

  final provider = this.provider;

  if (provider is EmailAuthProvider) {
    return EmailAuthFlow(
      provider: provider,
      action: widget.action,
      auth: widget.auth,
    );
  } else if (provider is EmailLinkAuthProvider) {
    return EmailLinkFlow(
      provider: provider,
      auth: widget.auth,
    );
  } else if (provider is OAuthProvider) {
    return OAuthFlow(
      provider: provider,
      action: widget.action,
      auth: widget.auth,
    );
  } else if (provider is PhoneAuthProvider) {
    return PhoneAuthFlow(
      provider: provider,
      action: widget.action,
      auth: widget.auth,
    );
  } else {
    throw Exception('Unknown provider $provider');
  }
}
```

### Default Provider Creation

When no provider is specified, the widget creates default providers based on the controller type:

```dart 222:231:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
AuthProvider _createDefaultProvider() {
  switch (T) {
    case EmailAuthController:
      return EmailAuthProvider();
    case PhoneAuthController:
      return PhoneAuthProvider();
    default:
      throw Exception("Can't create $T provider");
  }
}
```

### State Management

The widget uses a ValueListenableBuilder to react to AuthFlow state changes:

```dart 284:302:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
@override
Widget build(BuildContext context) {
  return AuthControllerProvider(
    action: flow.action,
    ctrl: flow,
    child: ValueListenableBuilder<AuthState>(
      valueListenable: flow,
      builder: (context, value, _) {
        final child = builder(
          context,
          value,
          flow as T,
          widget.child,
        );

        return AuthStateProvider(state: value, child: child);
      },
    ),
  );
}
```

## Usage Example

The documentation provides a comprehensive example of building a custom email sign-up form:

```dart 62:101:packages/firebase_ui_auth/lib/src/widgets/auth_flow_builder.dart
/// final emailCtrl = TextEditingController();
/// final passwordCtrl = TextEditingController();
///
/// AuthFlowBuilder<EmailAuthController>(
///   auth: fba.FirebaseAuth.instance,
///   action: AuthAction.signUp,
///   listener: (oldState, newState, ctrl) {
///     if (newState is UserCreated) {
///       Navigator.of(context).pushReplacementNamed('/profile');
///     }
///   },
///   builder: (context, state, ctrl, child) {
///     if (state is AwaitingEmailAndPassword) {
///       return Column(
///         children: [
///           TextField(
///             decoration: InputDecoration(labelText: 'Email'),
///             controller: emailCtrl,
///           ),
///           TextField(
///             decoration: InputDecoration(labelText: 'Password'),
///             controller: passwordCtrl,
///           ),
///           OutlinedButton(
///             child: Text('Sign Up'),
///             onPressed: () {
///               ctrl.setEmailAndPassword(emailCtrl.text, passwordCtrl.text);
///             }
///           ),
///         ]
///       );
///     } else if (state is SigningIn) {
///       return Center(child: CircularProgressIndicator());
///     } else if (state is AuthFailed) {
///       return ErrorText(exception: state.exception);
///     }
///   }
/// )
```

## Key Features

1. **Generic Type Safety**: Uses `T extends AuthController` to ensure type-safe controller access
2. **Flow Registry**: Static methods allow accessing flows across the widget tree using flow keys
3. **Automatic Flow Creation**: Creates appropriate AuthFlow instances based on provider types
4. **State Management**: Integrates with Flutter's ValueListenableBuilder for reactive updates
5. **Provider Pattern**: Uses AuthControllerProvider and AuthStateProvider to make auth state available to descendants
6. **Flexible UI**: Supports both builder functions and pre-built child widgets
7. **Lifecycle Management**: Properly handles flow initialization, updates, and disposal

## Supported Auth Flows

The widget supports four main authentication flows:

- EmailAuthFlow (email/password authentication)
- EmailLinkFlow (email link authentication)
- OAuthFlow (social login via OAuth providers)
- PhoneAuthFlow (phone number authentication)

Each flow has its own controller type and state machine, providing a consistent API while handling the specific requirements of each authentication method.
