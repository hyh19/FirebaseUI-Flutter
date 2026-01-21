# Email Authentication Provider

The `email_auth_provider.dart` file implements email and password authentication functionality within the FirebaseUI Auth package for Flutter. This provider enables users to sign in, sign up, and link accounts using traditional email/password credentials.

## EmailAuthListener

```dart 10:10:packages/firebase_ui_auth/lib/src/providers/email_auth_provider.dart
abstract class EmailAuthListener extends AuthListener {}
```

The `EmailAuthListener` is an abstract class that extends the base `AuthListener` class. It serves as a lifecycle listener for email authentication flows, providing callback hooks for different stages of the authentication process. While this class appears minimal, it inherits all the authentication lifecycle methods from its parent class, allowing implementers to respond to authentication events such as:

- `onBeforeSignIn()` - Called before authentication begins
- `onSignedIn()` - Called when authentication succeeds  
- `onError()` - Called when authentication fails

## EmailAuthProvider Class

```dart 15:16:packages/firebase_ui_auth/lib/src/providers/email_auth_provider.dart
class EmailAuthProvider
    extends AuthProvider<EmailAuthListener, fba.EmailAuthCredential> {
```

The `EmailAuthProvider` extends the generic `AuthProvider` class, specialized for email authentication. It uses `EmailAuthListener` for lifecycle callbacks and `fba.EmailAuthCredential` (Firebase Auth's email credential type) for authentication data.

### Provider Configuration

```dart 20:24:packages/firebase_ui_auth/lib/src/providers/email_auth_provider.dart
  @override
  final providerId = 'password';

  @override
  bool supportsPlatform(TargetPlatform platform) => true;
```

- **providerId**: Set to `'password'`, which corresponds to Firebase Auth's email/password authentication provider identifier
- **Platform Support**: Returns `true` for all platforms, indicating email authentication works across all supported Flutter platforms

### Sign Up Functionality

```dart 27:36:packages/firebase_ui_auth/lib/src/providers/email_auth_provider.dart
  void signUpWithCredential(fba.EmailAuthCredential credential) {
    authListener.onBeforeSignIn();
    auth
        .createUserWithEmailAndPassword(
          email: credential.email,
          password: credential.password!,
        )
        .then(authListener.onSignedIn)
        .catchError(authListener.onError);
  }
```

The `signUpWithCredential` method handles user registration:

1. Notifies listeners that authentication is about to begin (`onBeforeSignIn`)
2. Calls Firebase Auth's `createUserWithEmailAndPassword` with the email and password from the credential
3. Handles success by calling `onSignedIn` with the resulting user
4. Handles errors by calling `onError` with the exception

### Authentication Method

```dart 40:51:packages/firebase_ui_auth/lib/src/providers/email_auth_provider.dart
  void authenticate(
    String email,
    String password, [
    AuthAction action = AuthAction.signIn,
  ]) {
    final credential = fba.EmailAuthProvider.credential(
      email: email,
      password: password,
    ) as fba.EmailAuthCredential;

    onCredentialReceived(credential, action);
  }
```

The `authenticate` method is the primary entry point for email authentication:

1. Takes email, password, and an optional `AuthAction` (defaults to `signIn`)
2. Creates a Firebase Auth email credential from the provided email and password
3. Delegates to `onCredentialReceived` to handle the actual authentication flow based on the specified action

### Action-Based Credential Handling

```dart 54:76:packages/firebase_ui_auth/lib/src/providers/email_auth_provider.dart
  @override
  void onCredentialReceived(
    fba.EmailAuthCredential credential,
    AuthAction action,
  ) {
    switch (action) {
      case AuthAction.signIn:
        signInWithCredential(credential);
        break;
      case AuthAction.signUp:
        if (shouldUpgradeAnonymous) {
          return linkWithCredential(credential);
        }

        signUpWithCredential(credential);
        break;
      case AuthAction.link:
        linkWithCredential(credential);
        break;
      case AuthAction.none:
        super.onCredentialReceived(credential, action);
        break;
    }
  }
```

The `onCredentialReceived` method routes authentication based on the desired action:

- **signIn**: Calls `signInWithCredential` (inherited from parent class) for user login
- **signUp**:
  - If upgrading an anonymous user, calls `linkWithCredential` to link the email credential
  - Otherwise, calls `signUpWithCredential` for new user registration
- **link**: Calls `linkWithCredential` (inherited) to add email authentication to an existing account
- **none**: Delegates to parent class for default handling

## Integration with FirebaseUI Auth

This provider integrates seamlessly with the broader FirebaseUI Auth system:

- Extends the base `AuthProvider` class, ensuring compatibility with the authentication flow framework
- Uses Firebase Auth's native email/password authentication under the hood
- Supports all standard authentication actions (sign in, sign up, account linking)
- Provides lifecycle callbacks through the `EmailAuthListener` interface
- Handles anonymous user upgrades automatically when signing up

The provider abstracts away the complexity of Firebase Auth's email/password authentication while maintaining full compatibility with Firebase's security features and best practices.
