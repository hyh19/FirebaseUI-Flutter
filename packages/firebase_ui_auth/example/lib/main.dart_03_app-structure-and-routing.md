# main.dart - App Structure and Routing

## Overview

This document explains the main application structure and routing logic in the Firebase UI Auth example app. The `FirebaseAuthUIExample` class serves as the root widget and manages navigation between different authentication screens.

## FirebaseAuthUIExample Class

The main application widget extends `StatelessWidget` and provides routing logic:

```dart 66:77:lib/main.dart
class FirebaseAuthUIExample extends StatelessWidget {
  const FirebaseAuthUIExample({super.key});

  String get initialRoute {
    final user = FirebaseAuth.instance.currentUser;

    return switch (user) {
      null => '/',
      User(emailVerified: false, email: final String _) => '/verify-email',
      _ => '/profile',
    };
  }
```

### Initial Route Logic

The app determines the initial route based on the user's authentication state:

- **Unauthenticated (`null`)**: Routes to sign-in screen (`/`)
- **Unverified Email**: Routes to email verification screen (`/verify-email`)
- **Verified User**: Routes to profile screen (`/profile`)

This ensures users are directed to the appropriate screen based on their authentication status.

## MaterialApp Configuration

The app configures Material Design theme and routing:

```dart 80:112:lib/main.dart
  @override
  Widget build(BuildContext context) {
    final buttonStyle = ButtonStyle(
      padding: WidgetStateProperty.all(const EdgeInsets.all(12)),
      shape: WidgetStateProperty.all(
        RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
      ),
    );

    return MaterialApp(
      theme: ThemeData(
        brightness: Brightness.light,
        visualDensity: VisualDensity.standard,
        useMaterial3: true,
        inputDecorationTheme: const InputDecorationTheme(
          border: OutlineInputBorder(),
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(style: buttonStyle),
        textButtonTheme: TextButtonThemeData(style: buttonStyle),
        outlinedButtonTheme: OutlinedButtonThemeData(style: buttonStyle),
      ),
      initialRoute: initialRoute,
      routes: {
        // Route definitions...
      },
      title: 'Firebase UI demo',
      debugShowCheckedModeBanner: false,
      supportedLocales: const [Locale('en')],
      localizationsDelegates: [
        FirebaseUILocalizations.withDefaultOverrides(const LabelOverrides()),
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        FirebaseUILocalizations.delegate,
      ],
    );
  }
```

### Theme Configuration

- **Material 3**: Uses the latest Material Design 3 specifications
- **Light Theme**: Configured for light mode (brightness: Brightness.light)
- **Button Styling**: Consistent rounded button styles across all button types
- **Input Decoration**: Outline borders for text input fields

### App Properties

- **Title**: "Firebase UI demo" for window/app title
- **Debug Banner**: Disabled for cleaner demo appearance
- **Localization**: Configured with Firebase UI localizations and custom overrides

## Route Definitions

The app defines several routes for different authentication flows:

### 1. Root Route (`/`)

The sign-in screen with comprehensive authentication options:

```dart 115:190:lib/main.dart
        '/': (context) {
          final platform = Theme.of(context).platform;

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
                child: Text('Welcome to Firebase UI! $actionText.');
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
        },
```

### 2. Email Verification Route (`/verify-email`)

```dart 192:206:lib/main.dart
        '/verify-email': (context) {
          return EmailVerificationScreen(
            headerBuilder: headerIcon(Icons.verified),
            sideBuilder: sideIcon(Icons.verified),
            actionCodeSettings: actionCodeSettings,
            actions: [
              EmailVerifiedAction(() {
                Navigator.pushReplacementNamed(context, '/profile');
              }),
              AuthCancelledAction((context) {
                FirebaseUIAuth.signOut(context: context);
                Navigator.pushReplacementNamed(context, '/');
              }),
            ],
          );
        },
```

### 3. Phone Input Route (`/phone`)

```dart 208:224:lib/main.dart
        '/phone': (context) {
          return PhoneInputScreen(
            actions: [
              SMSCodeRequestedAction((context, action, flowKey, phone) {
                Navigator.of(context).pushReplacementNamed(
                  '/sms',
                  arguments: {
                    'action': action,
                    'flowKey': flowKey,
                    'phone': phone,
                  },
                );
              }),
            ],
            headerBuilder: headerIcon(Icons.phone),
            sideBuilder: sideIcon(Icons.phone),
          );
        },
```

### 4. SMS Code Input Route (`/sms`)

```dart 226:241:lib/main.dart
        '/sms': (context) {
          final arguments = ModalRoute.of(context)?.settings.arguments
              as Map<String, dynamic>?;

          return SMSCodeInputScreen(
            actions: [
              AuthStateChangeAction<SignedIn>((context, state) {
                Navigator.of(context).pushReplacementNamed('/profile');
              })
            ],
            flowKey: arguments?['flowKey'],
            action: arguments?['action'],
            headerBuilder: headerIcon(Icons.sms_outlined),
            sideBuilder: sideIcon(Icons.sms_outlined),
          );
        },
```

### 5. Forgot Password Route (`/forgot-password`)

```dart 242:251:lib/main.dart
        '/forgot-password': (context) {
          final arguments = ModalRoute.of(context)?.settings.arguments
              as Map<String, dynamic>?;

          return ForgotPasswordScreen(
            email: arguments?['email'],
            headerMaxExtent: 200,
            headerBuilder: headerIcon(Icons.lock),
            sideBuilder: sideIcon(Icons.lock),
          );
        },
```

### 6. Email Link Sign-In Route (`/email-link-sign-in`)

```dart 253:264:lib/main.dart
        '/email-link-sign-in': (context) {
          return EmailLinkSignInScreen(
            actions: [
              AuthStateChangeAction<SignedIn>((context, state) {
                Navigator.pushReplacementNamed(context, '/profile');
              }),
            ],
            provider: emailLinkProviderConfig,
            headerMaxExtent: 200,
            headerBuilder: headerIcon(Icons.link),
            sideBuilder: sideIcon(Icons.link),
          );
        },
```

### 7. Profile Route (`/profile`)

```dart 266:283:lib/main.dart
        '/profile': (context) {
          final platform = Theme.of(context).platform;

          return ProfileScreen(
            actions: [
              SignedOutAction((context) {
                Navigator.pushReplacementNamed(context, '/');
              }),
              mfaAction,
            ],
            actionCodeSettings: actionCodeSettings,
            showMFATile: kIsWeb ||
                platform == TargetPlatform.iOS ||
                platform == TargetPlatform.android,
            showUnlinkConfirmationDialog: true,
            showDeleteConfirmationDialog: true,
          );
        },
```

## MFA Action Configuration

The app includes Multi-Factor Authentication (MFA) handling:

```dart 88:99:lib/main.dart
    final mfaAction = AuthStateChangeAction<MFARequired>(
      (context, state) async {
        final nav = Navigator.of(context);

        await startMFAVerification(
          resolver: state.resolver,
          context: context,
        );

        nav.pushReplacementNamed('/profile');
      },
    );
```

This action handles MFA challenges by starting the verification process and redirecting to the profile screen upon completion.

## Screen Features

### Common Screen Elements

- **Header Builders**: Custom headers using images or icons
- **Side Builders**: Side panel content for larger screens
- **Action Handlers**: Various authentication state change actions
- **Navigation Logic**: Proper routing between authentication states

### Platform-Specific Features

- **iOS Tracking**: App Tracking Transparency card shown only on iOS
- **MFA Support**: Multi-factor authentication available on web, iOS, and Android
- **Confirmation Dialogs**: Account deletion and provider unlinking confirmations

## Navigation Patterns

The app uses several navigation patterns:

- **pushNamed**: Navigate to new routes (adds to navigation stack)
- **pushReplacementNamed**: Replace current route (removes from stack)
- **Arguments Passing**: Route arguments for passing data between screens
- **State-Based Routing**: Initial route determined by authentication state
