# main.dart - Imports and Configuration

## Overview

This document explains the imports and initial configuration setup in the Firebase UI Auth example app. This section establishes the foundation for Firebase authentication and UI components.

## Imports Section

The file begins with a comprehensive set of imports that bring in all necessary Firebase and Flutter packages:

```dart 1:22:lib/main.dart
// Copyright 2022, the Chromium project authors.  Please see the AUTHORS file
// for details. All rights reserved. Use of this source code is governed by a
// BSD-style license that can be found in the LICENSE file.

import 'package:firebase_auth/firebase_auth.dart'
    hide PhoneAuthProvider, EmailAuthProvider;
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_ui_auth/firebase_ui_auth.dart';
import 'package:firebase_ui_localizations/firebase_ui_localizations.dart';
import 'package:firebase_ui_oauth_apple/firebase_ui_oauth_apple.dart';
import 'package:firebase_ui_oauth_facebook/firebase_ui_oauth_facebook.dart';
import 'package:firebase_ui_oauth_google/firebase_ui_oauth_google.dart';
import 'package:firebase_ui_oauth_twitter/firebase_ui_oauth_twitter.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:app_tracking_transparency/app_tracking_transparency.dart';

import 'config.dart';
import 'decorations.dart';
import 'firebase_options.dart';
```

### Key Import Details

- **Firebase Auth**: Uses `hide` to avoid conflicts with Firebase UI's auth providers
- **Firebase UI Packages**: Imports the main UI package plus OAuth providers for Apple, Facebook, Google, and Twitter
- **Flutter Framework**: Core Flutter packages for foundation, material design, and localization
- **App Tracking Transparency**: iOS-specific package for tracking permission handling
- **Local Files**: Imports configuration, decorations, and Firebase options

## Action Code Settings Configuration

The app configures email link authentication settings:

```dart 23:32:lib/main.dart
final actionCodeSettings = ActionCodeSettings(
  url: 'https://flutterfire-e2e-tests.firebaseapp.com',
  handleCodeInApp: true,
  androidMinimumVersion: '1',
  androidPackageName: 'io.flutter.plugins.firebase_ui_example',
  iOSBundleId: 'io.flutter.plugins.fireabaseUiExample',
);
final emailLinkProviderConfig = EmailLinkAuthProvider(
  actionCodeSettings: actionCodeSettings,
);
```

### Configuration Details

- **URL**: Firebase app URL for handling email links
- **Handle Code in App**: Processes authentication links within the app instead of browser
- **Platform Specific**: Different package/bundle IDs for Android and iOS
- **Email Link Provider**: Configures the email link authentication provider

## Main Function

The main entry point initializes Firebase and configures authentication providers:

```dart 34:54:lib/main.dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  // await FirebaseAuth.instance.useAuthEmulator('localhost', 9099);

  FirebaseUIAuth.configureProviders([
    EmailAuthProvider(),
    emailLinkProviderConfig,
    PhoneAuthProvider(),
    GoogleProvider(clientId: GOOGLE_CLIENT_ID),
    AppleProvider(),
    FacebookProvider(clientId: FACEBOOK_CLIENT_ID),
    TwitterProvider(
      apiKey: TWITTER_API_KEY,
      apiSecretKey: TWITTER_API_SECRET_KEY,
      redirectUri: TWITTER_REDIRECT_URI,
    ),
  ]);

  runApp(const FirebaseAuthUIExample());
}
```

### Initialization Steps

1. **Flutter Binding**: Ensures Flutter framework is initialized before async operations
2. **Firebase Initialization**: Sets up Firebase with platform-specific options
3. **Auth Emulator**: Commented out option for local development testing
4. **Provider Configuration**: Registers all supported authentication methods
5. **App Launch**: Starts the main application widget

## Authentication Providers

The app supports multiple authentication methods:

- **Email/Password**: Standard email and password authentication
- **Email Link**: Passwordless authentication via email links
- **Phone**: SMS-based authentication
- **OAuth Providers**: Google, Apple, Facebook, and Twitter social login

Each provider is configured with appropriate credentials from the config file.
