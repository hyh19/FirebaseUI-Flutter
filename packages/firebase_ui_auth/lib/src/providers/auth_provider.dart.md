# Auth Provider

This file defines the core authentication provider interface and shared logic for Firebase UI Auth. It provides the foundation for implementing different authentication methods (email/password, OAuth providers, phone authentication, etc.) in a consistent way.

## Overview

The file contains three main components:

1. A default error handler for authentication errors
2. An abstract `AuthListener` interface for authentication lifecycle callbacks
3. An abstract `AuthProvider` class that implements shared authentication logic

## Default Error Handler

```dart 9:22:packages/firebase_ui_auth/lib/src/providers/auth_provider.dart
/// Default error handler that starts MFA flow
/// if [FirebaseAuthMultiFactorException] is thrown.
void defaultOnAuthError(AuthProvider provider, Object error) {
  if (error is! fba.FirebaseAuthException) {
    throw error;
  }

  if (error is fba.FirebaseAuthMultiFactorException) {
    provider.authListener.onMFARequired(error.resolver);
    return;
  }

  throw error;
}
```

The `defaultOnAuthError` function provides a standard way to handle authentication errors. It specifically checks for `FirebaseAuthMultiFactorException` (MFA/multi-factor authentication errors) and triggers the MFA flow by calling `onMFARequired` on the auth listener. All other errors are re-thrown.

This function is used by auth providers to handle errors that occur during the authentication process, ensuring that MFA challenges are properly handled.

## AuthListener Interface

```dart 24:58:packages/firebase_ui_auth/lib/src/providers/auth_provider.dart
/// An interface that describes authentication process lifecycle.
///
/// See implementers:
/// - [EmailAuthListener]
/// - [EmailLinkAuthListener]
/// - [PhoneAuthListener]
abstract class AuthListener {
  /// Current [AuthProvider] that is being used to authenticate the user.
  AuthProvider get provider;

  /// {@macro ui.auth.auth_controller.auth}
  fba.FirebaseAuth get auth;

  /// Called if an error occured during the authentication process.
  void onError(Object error);

  /// Called right before the authentication process starts.
  void onBeforeSignIn();

  /// Called if the user has successfully signed in.
  void onSignedIn(fba.UserCredential credential);

  /// Called before an attempt to link the credential with currently signed in
  /// user account.
  void onCredentialReceived(fba.AuthCredential credential);

  /// Called if the credential was successfully linked with the user account.
  void onCredentialLinked(fba.AuthCredential credential);

  /// Called when the user cancels the sign in process.
  void onCanceled();

  /// Called when the user has to complete MFA.
  void onMFARequired(fba.MultiFactorResolver resolver);
}
```

The `AuthListener` abstract class defines the interface that all authentication listeners must implement. It provides lifecycle callbacks for different stages of the authentication process:

- **`provider`** and **`auth`**: Provide access to the current auth provider and Firebase Auth instance
- **`onError`**: Called when any error occurs during authentication
- **`onBeforeSignIn`**: Called just before the sign-in process begins
- **`onSignedIn`**: Called when the user successfully signs in with their credentials
- **`onCredentialReceived`**: Called when a credential is obtained and about to be used
- **`onCredentialLinked`**: Called when a credential is successfully linked to an existing account
- **`onCanceled`**: Called when the user cancels the authentication process
- **`onMFARequired`**: Called when multi-factor authentication is required

Different auth providers (email, phone, OAuth) have their own implementations of this interface to handle provider-specific logic.

## AuthProvider Base Class

```dart 60:154:packages/firebase_ui_auth/lib/src/providers/auth_provider.dart
/// {@template ui.auth.auth_provider}
/// An interface that all auth providers should implement.
/// Contains shared authentication logic.
/// {@endtemplate}
abstract class AuthProvider<T extends AuthListener,
    K extends fba.AuthCredential> {
  /// {@macro ui.auth.auth_controller.auth}
  late fba.FirebaseAuth auth;

  /// {@template ui.auth.auth_provider.auth_listener}
  /// An instance of the [AuthListener] that is used to notify about the
  /// current state of the authentication process.
  /// {@endtemplate}
  T get authListener;

  /// {@macro ui.auth.auth_provider.auth_listener}
  set authListener(T listener);

  /// {@template ui.auth.auth_provider.provider_id}
  /// String identifer of the auth provider, for example: `'password'`,
  /// `'phone'` or `'google.com'`.
  /// {@endtemplate}
  String get providerId;

  /// Verifies that an [AuthProvider] is supported on a [platform].
  bool supportsPlatform(TargetPlatform platform);

  /// {@macro ui.auth.auth_provider}
  AuthProvider();

  /// Indicates whether the user should be upgraded and new credential should be
  /// linked.
  bool get shouldUpgradeAnonymous => auth.currentUser?.isAnonymous ?? false;
```

The `AuthProvider` class is a generic abstract base class that all authentication providers extend. It uses two type parameters:

- `T`: The specific `AuthListener` implementation (must extend `AuthListener`)
- `K`: The type of `AuthCredential` used by this provider (must extend `fba.AuthCredential`)

### Key Properties

- **`auth`**: Firebase Auth instance used for authentication operations
- **`authListener`**: The listener that receives lifecycle callbacks during authentication
- **`providerId`**: String identifier for the auth provider (e.g., 'password', 'phone', 'google.com')
- **`supportsPlatform`**: Method to check if the provider works on a specific platform
- **`shouldUpgradeAnonymous`**: Computed property that checks if the current user is anonymous and should be upgraded

### Core Authentication Methods

```dart 94:101:packages/firebase_ui_auth/lib/src/providers/auth_provider.dart
  /// Signs the user in with the provided [AuthCredential].
  void signInWithCredential(K credential) {
    authListener.onBeforeSignIn();
    auth
        .signInWithCredential(credential)
        .then(authListener.onSignedIn)
        .catchError(authListener.onError);
  }
```

The `signInWithCredential` method handles the basic sign-in flow:

1. Notifies the listener that sign-in is about to start (`onBeforeSignIn`)
2. Calls Firebase Auth's `signInWithCredential` method
3. Notifies the listener of success (`onSignedIn`) or failure (`onError`)

```dart 103:117:packages/firebase_ui_auth/lib/src/providers/auth_provider.dart
  /// Links a provided [AuthCredential] with the currently signed in user
  /// account.
  void linkWithCredential(K credential) {
    authListener.onCredentialReceived(credential);

    try {
      final user = auth.currentUser!;
      user
          .linkWithCredential(credential)
          .then((_) => authListener.onCredentialLinked(credential))
          .catchError(authListener.onError);
    } catch (err) {
      authListener.onError(err);
    }
  }
```

The `linkWithCredential` method handles linking additional credentials to an existing account:

1. Notifies the listener that a credential was received (`onCredentialReceived`)
2. Gets the current user (throws if no user is signed in)
3. Calls the user's `linkWithCredential` method
4. Notifies the listener of success (`onCredentialLinked`) or failure (`onError`)

### Main Credential Handler

```dart 119:153:packages/firebase_ui_auth/lib/src/providers/auth_provider.dart
  /// {@template ui.auth.auth_provider.on_credential_received}
  /// A method that is called when the user has successfully completed the
  /// authentication process and decides what to do with the obtained
  /// [credential].
  ///
  /// [linkWithCredential] and respectful lifecycle hooks are called if [action]
  /// is [AuthAction.link].
  ///
  /// [signInWithCredential] and respectful lifecycle hooks are called
  /// if [action] is [AuthAction.signIn].
  ///
  /// [FirebaseAuth.createUserWithEmailAndPassword] and respectful lifecycle
  /// hooks are called if action is [AuthAction.signUp].
  /// {@endtemplate}
  void onCredentialReceived(K credential, AuthAction action) {
    switch (action) {
      case AuthAction.link:
        linkWithCredential(credential);
        break;
      case AuthAction.signIn:
      // Only email provider has a different action for sign in and sign up
      // and implements it's own sign up logic.
      case AuthAction.signUp:
        if (shouldUpgradeAnonymous) {
          linkWithCredential(credential);
          break;
        }

        signInWithCredential(credential);
        break;
      case AuthAction.none:
        authListener.onCredentialReceived(credential);
        break;
    }
  }
```

The `onCredentialReceived` method is the main entry point that decides what to do with a credential based on the requested `AuthAction`:

- **`AuthAction.link`**: Links the credential to the current account
- **`AuthAction.signIn`** or **`AuthAction.signUp`**:
  - If the current user is anonymous, upgrades them by linking the credential
  - Otherwise, signs in with the credential
- **`AuthAction.none`**: Just notifies the listener that a credential was received (no action taken)

Note that sign-in and sign-up actions are handled the same way, except the email provider implements its own sign-up logic separately.

## Architecture Patterns

This file establishes several important patterns for the Firebase UI Auth library:

1. **Provider Pattern**: Each auth method (email, phone, OAuth) implements `AuthProvider`
2. **Observer Pattern**: `AuthListener` provides lifecycle callbacks for UI updates
3. **Strategy Pattern**: Different providers can be swapped while using the same interface
4. **Template Method Pattern**: `AuthProvider` provides common authentication logic that subclasses customize

## Usage Context

This file is used as the foundation for all authentication providers in Firebase UI Auth. Concrete implementations include:

- Email/password authentication
- Phone number authentication  
- OAuth providers (Google, Facebook, Apple, Twitter)
- Email link authentication

Each provider extends `AuthProvider` and implements provider-specific logic while inheriting the shared authentication flow and error handling.
