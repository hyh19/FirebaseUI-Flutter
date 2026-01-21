# AuthFlow Class Documentation

## Overview

The `auth_flow.dart` file contains the core authentication flow implementation for Firebase UI Auth. It defines the base `AuthFlow` class that manages the authentication state and provides shared logic for different authentication providers.

## Key Components

### AuthCancelledException

```dart 15:19:packages/firebase_ui_auth/lib/src/auth_flow.dart
class AuthCancelledException implements Exception {
  AuthCancelledException([this.message = 'User has cancelled auth']);

  final String message;
}
```

A simple exception class thrown when the user cancels the authentication process. It implements the standard `Exception` interface and provides a default cancellation message.

## AuthFlow Class

### Class Declaration

```dart 34:35:packages/firebase_ui_auth/lib/src/auth_flow.dart
class AuthFlow<T extends AuthProvider> extends ValueNotifier<AuthState>
    implements AuthController, AuthListener {
```

The `AuthFlow` is a generic class that extends `ValueNotifier<AuthState>` and implements both `AuthController` and `AuthListener` interfaces. The generic type `T` must extend `AuthProvider`, allowing different authentication providers (email, OAuth, phone, etc.) to be used with the same flow logic.

### Key Properties

#### FirebaseAuth Instance

```dart 37:37:packages/firebase_ui_auth/lib/src/auth_flow.dart
fba.FirebaseAuth auth;
```

The Firebase Auth instance used for authentication operations. Defaults to `FirebaseAuth.instance` if not provided.

#### Initial State

```dart 41:41:packages/firebase_ui_auth/lib/src/auth_flow.dart
final AuthState initialState;
```

The starting state of the authentication flow. Different auth flows may have different initial states.

#### Auth Provider

```dart 45:45:packages/firebase_ui_auth/lib/src/auth_flow.dart
final T _provider;
```

The private provider instance that handles the actual authentication logic. The getter ensures the provider is properly configured with the auth listener.

```dart 47:48:packages/firebase_ui_auth/lib/src/auth_flow.dart
T get provider => _provider..authListener = this;
```

#### Auth Action

```dart 51:61:packages/firebase_ui_auth/lib/src/auth_flow.dart
AuthAction get action {
  if (_action != null) {
    return _action!;
  }

  if (auth.currentUser != null) {
    return AuthAction.link;
  }

  return AuthAction.signIn;
}
```

Determines the current authentication action. If explicitly set, returns that value. Otherwise, automatically determines:

- `AuthAction.link` if a user is currently signed in (linking additional credentials)
- `AuthAction.signIn` if no user is signed in (new sign-in)

### Constructor

```dart 89:109:packages/firebase_ui_auth/lib/src/auth_flow.dart
AuthFlow({
  required this.initialState,
  required T provider,
  fba.FirebaseAuth? auth,
  AuthAction? action,
})  : auth = auth ?? fba.FirebaseAuth.instance,
      _action = action,
      _provider = provider,
      super(initialState) {
  _provider.authListener = this;
  _provider.auth = auth ?? fba.FirebaseAuth.instance;
}
```

Initializes the auth flow with:

- Required `initialState` and `provider`
- Optional `auth` instance (defaults to FirebaseAuth.instance)
- Optional explicit `action` override
- Sets up the provider with the auth listener and instance

### AuthListener Implementation

The class implements `AuthListener` interface methods that respond to authentication events:

#### Credential Reception

```dart 112:114:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onCredentialReceived(fba.AuthCredential credential) {
  value = CredentialReceived(credential);
}
```

Called when authentication credentials are received, updates the state to `CredentialReceived`.

#### Pre-Sign In

```dart 117:119:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onBeforeSignIn() {
  value = const SigningIn();
}
```

Called before the sign-in process begins, sets the state to `SigningIn`.

#### Credential Linking

```dart 122:124:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onCredentialLinked(fba.AuthCredential credential) {
  value = CredentialLinked(credential, auth.currentUser!);
}
```

Called when credentials are successfully linked to an existing user account.

#### Successful Sign In

```dart 127:133:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onSignedIn(fba.UserCredential credential) {
  if (credential.additionalUserInfo?.isNewUser ?? false) {
    value = UserCreated(credential);
  } else {
    value = SignedIn(credential.user);
  }
}
```

Handles successful sign-in by checking if this is a new user or returning user, setting appropriate states.

#### Error Handling

```dart 142:150:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onError(Object error) {
  try {
    defaultOnAuthError(provider, error);
  } on AuthCancelledException {
    reset();
  } on Exception catch (err) {
    value = AuthFailed(err);
  }
}
```

Processes authentication errors. Special handling for `AuthCancelledException` (resets the flow) and other exceptions (sets failed state).

#### Cancellation

```dart 153:155:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onCanceled() {
  value = initialState;
}
```

Resets the authentication state to the initial state when cancelled.

#### Multi-Factor Authentication

```dart 158:160:packages/firebase_ui_auth/lib/src/auth_flow.dart
void onMFARequired(fba.MultiFactorResolver resolver) {
  value = MFARequired(resolver);
}
```

Handles multi-factor authentication requirements by setting the appropriate state.

### Cleanup and Disposal

```dart 75:81:packages/firebase_ui_auth/lib/src/auth_flow.dart
VoidCallback get onDispose {
  return () {
    for (var callback in _onDispose) {
      callback();
    }
  };
}
```

The `onDispose` callback executes all registered cleanup functions when the auth flow completes.

```dart 84:86:packages/firebase_ui_auth/lib/src/auth_flow.dart
set onDispose(VoidCallback callback) {
  _onDispose.add(callback);
}
```

Allows registering cleanup callbacks that will be executed when the flow is disposed.

### Reset Functionality

```dart 136:139:packages/firebase_ui_auth/lib/src/auth_flow.dart
void reset() {
  value = initialState;
  onDispose();
}
```

Resets the auth flow to its initial state and executes all cleanup callbacks.

## Usage Context

This base `AuthFlow` class is designed to be extended by specific authentication flow implementations:

- `EmailAuthFlow` - for email/password authentication
- `EmailLinkFlow` - for email link authentication  
- `OAuthFlow` - for OAuth provider authentication
- `PhoneAuthFlow` - for phone number authentication

The class provides the shared state management and event handling logic, while subclasses implement provider-specific authentication logic.

## State Management

The class uses Flutter's `ValueNotifier` pattern to manage authentication state. Different `AuthState` subclasses represent various stages:

- `Uninitialized` - Initial state
- `SigningIn` - Authentication in progress
- `CredentialReceived` - Credentials obtained
- `SignedIn` - Successful authentication for existing user
- `UserCreated` - Successful authentication for new user
- `CredentialLinked` - Additional credentials linked
- `AuthFailed` - Authentication error occurred
- `MFARequired` - Multi-factor authentication needed

Widgets can listen to state changes using `ValueListenableBuilder` or similar patterns to update the UI accordingly.

## Integration with AuthFlowBuilder

The `AuthFlow` class is designed to work with `AuthFlowBuilder` widget, which provides the UI integration layer. The builder widget manages the lifecycle of auth flows and handles the widget tree integration.
