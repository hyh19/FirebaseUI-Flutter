# SignInScreen Configuration in Firebase UI Auth

This section of `main.dart` (lines 118-190) demonstrates how to configure a comprehensive Firebase UI Auth `SignInScreen` widget with multiple authentication providers, custom actions, and dynamic UI builders.

## Overview

The `SignInScreen` is the main authentication interface that handles user sign-in and sign-up flows. It's configured with various actions that respond to different authentication states and user interactions, along with custom styling and UI builders that adapt based on the current authentication action (sign-in vs sign-up).

## SignInScreen Widget Configuration

```dart 118:190:lib/main.dart
return SignInScreen(
  actions: [
    ForgotPasswordAction((context, email) {
      Navigator.pushNamed(
        context,
        '/forgot-password',
        arguments: {'email': email},
      );
    }),
    VerifyPhoneAction((context, _) {
      Navigator.pushNamed(context, '/phone');
    }),
    AuthStateChangeAction((context, state) {
      final user = switch (state) {
        SignedIn(user: final user) => user,
        CredentialLinked(user: final user) => user,
        UserCreated(credential: final cred) => cred.user,
        _ => null,
      };

      switch (user) {
        case User(emailVerified: true):
          Navigator.pushReplacementNamed(context, '/profile');
        case User(emailVerified: false, email: final String _):
          Navigator.pushNamed(context, '/verify-email');
      }
    }),
    mfaAction,
    EmailLinkSignInAction((context) {
      Navigator.pushReplacementNamed(context, '/email-link-sign-in');
    }),
  ],
  styles: const {
    EmailFormStyle(signInButtonVariant: ButtonVariant.filled),
  },
  headerBuilder: headerImage('assets/images/flutterfire_logo.png'),
  sideBuilder: sideImage('assets/images/flutterfire_logo.png'),
  subtitleBuilder: (context, action) {
    final actionText = switch (action) {
      AuthAction.signIn => 'Please sign in to continue.',
      AuthAction.signUp => 'Please create an account to continue',
      _ => throw Exception('Invalid action: $action'),
    };

    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text('Welcome to Firebase UI! $actionText.'),
    );
  },
  footerBuilder: (context, action) {
    final actionText = switch (action) {
      AuthAction.signIn => 'signing in',
      AuthAction.signUp => 'registering',
      _ => throw Exception('Invalid action: $action'),
    };

    return Column(
      children: [
        if (platform == TargetPlatform.iOS)
          const AppTrackingTransparencyCard(),
        Center(
          child: Padding(
            padding: const EdgeInsets.only(top: 16),
            child: Text(
              'By $actionText, you agree to our terms and conditions.',
              style: const TextStyle(color: Colors.grey),
            ),
          ),
        ),
      ],
    );
  },
);
```

## Actions Configuration

The `actions` parameter defines how the screen responds to various authentication events and user interactions:

### 1. ForgotPasswordAction

```dart 120:126:lib/main.dart
ForgotPasswordAction((context, email) {
  Navigator.pushNamed(
    context,
    '/forgot-password',
    arguments: {'email': email},
  );
}),
```

Handles the "Forgot Password" functionality. When triggered, it navigates to the `/forgot-password` route and passes the user's email address as an argument for pre-filling the reset form.

### 2. VerifyPhoneAction

```dart 127:129:lib/main.dart
VerifyPhoneAction((context, _) {
  Navigator.pushNamed(context, '/phone');
}),
```

Triggers when a user needs to verify their phone number. It navigates to the phone input screen (`/phone`) where users can enter their phone number for SMS verification.

### 3. AuthStateChangeAction

```dart 130:144:lib/main.dart
AuthStateChangeAction((context, state) {
  final user = switch (state) {
    SignedIn(user: final user) => user,
    CredentialLinked(user: final user) => user,
    UserCreated(credential: final cred) => cred.user,
    _ => null,
  };

  switch (user) {
    case User(emailVerified: true):
      Navigator.pushReplacementNamed(context, '/profile');
    case User(emailVerified: false, email: final String _):
      Navigator.pushNamed(context, '/verify-email');
  }
}),
```

This is the core authentication state handler that responds to various auth state changes:

- **SignedIn**: User successfully signed in with an existing account
- **CredentialLinked**: User linked additional credentials to their account
- **UserCreated**: New user account was created

The logic then checks the user's email verification status:

- If email is verified → Navigate to profile screen
- If email is not verified → Navigate to email verification screen

### 4. MFA Action

```dart 145:145:lib/main.dart
mfaAction,
```

References the `mfaAction` variable defined earlier in the file (lines 88-99). This handles Multi-Factor Authentication (MFA) requirements when additional verification is needed.

### 5. EmailLinkSignInAction

```dart 146:148:lib/main.dart
EmailLinkSignInAction((context) {
  Navigator.pushReplacementNamed(context, '/email-link-sign-in');
}),
```

Handles email link sign-in flow, navigating to a dedicated screen for email link authentication.

## Styles Configuration

```dart 150:152:lib/main.dart
styles: const {
  EmailFormStyle(signInButtonVariant: ButtonVariant.filled),
},
```

Customizes the appearance of authentication forms. Here, the email form's sign-in button is configured to use a filled variant (solid background) rather than the default outlined style.

## UI Builders

The screen uses several builder functions to customize its appearance dynamically based on the current authentication action.

### Header Builder

```dart 153:153:lib/main.dart
headerBuilder: headerImage('assets/images/flutterfire_logo.png'),
```

Displays the FlutterFire logo at the top of the screen. The `headerImage` function (likely defined in `decorations.dart`) creates a consistent header image across different screens.

### Side Builder

```dart 154:154:lib/main.dart
sideBuilder: sideImage('assets/images/flutterfire_logo.png'),
```

Shows the logo on the side of the screen (typically on larger screens or landscape orientation).

### Subtitle Builder

```dart 155:166:lib/main.dart
subtitleBuilder: (context, action) {
  final actionText = switch (action) {
    AuthAction.signIn => 'Please sign in to continue.',
    AuthAction.signUp => 'Please create an account to continue',
    _ => throw Exception('Invalid action: $action'),
  };

  return Padding(
    padding: const EdgeInsets.only(bottom: 8),
    child: Text('Welcome to Firebase UI! $actionText.'),
  );
},
```

Creates dynamic subtitles that change based on whether the user is signing in or signing up. Uses modern Dart pattern matching to determine the appropriate text.

### Footer Builder

```dart 167:189:lib/main.dart
footerBuilder: (context, action) {
  final actionText = switch (action) {
    AuthAction.signIn => 'signing in',
    AuthAction.signUp => 'registering',
    _ => throw Exception('Invalid action: $action'),
  };

  return Column(
    children: [
      if (platform == TargetPlatform.iOS)
        const AppTrackingTransparencyCard(),
      Center(
        child: Padding(
          padding: const EdgeInsets.only(top: 16),
          child: Text(
            'By $actionText, you agree to our terms and conditions.',
            style: const TextStyle(color: Colors.grey),
          ),
        ),
      ),
    ],
  );
},
```

Builds the footer content with platform-specific elements:

- **iOS Only**: Shows an App Tracking Transparency card for iOS users
- **All Platforms**: Displays terms and conditions text that adapts to the current action

## Authentication Flow Logic

The configuration creates a comprehensive authentication flow:

1. **Entry Point**: Users land on this screen for initial authentication
2. **Provider Selection**: Firebase UI automatically shows configured providers (email, phone, Google, Apple, etc.)
3. **Action-Based Responses**: Different user actions trigger appropriate navigation
4. **State Management**: Authentication state changes determine the next screen
5. **Email Verification**: Unverified emails redirect to verification flow
6. **Platform Adaptation**: UI adapts based on platform (iOS tracking permissions)

## Key Design Patterns

### Pattern Matching

The code extensively uses Dart's modern pattern matching (`switch` expressions) for:

- Determining authentication states
- Checking user email verification status
- Creating action-specific text

### Platform-Specific UI

The footer builder demonstrates platform-aware UI construction, showing iOS-specific tracking permission controls only on iOS devices.

### Navigation with Arguments

Several actions pass data between screens using Flutter's named routes with arguments, enabling seamless flow between different authentication screens.

### Reusable Actions

The `mfaAction` is defined once and reused across multiple screens, demonstrating the DRY (Don't Repeat Yourself) principle.

This configuration showcases Firebase UI Auth's flexibility in handling complex authentication flows while maintaining clean, readable code through modern Dart features.
