# ProfileScreen Configuration in main.dart

## Overview

The ProfileScreen configuration in the `/profile` route demonstrates how to set up a comprehensive user profile management screen using Firebase UI Auth. This configuration includes user actions, multi-factor authentication (MFA) handling, email verification settings, and platform-specific UI features.

## ProfileScreen Widget Structure

```dart 269:282:lib/main.dart
ProfileScreen(
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
)
```

## Key Configuration Parameters

### actions

An array of `AuthStateChangeAction` objects that define how the ProfileScreen responds to various authentication state changes:

1. **SignedOutAction**: Handles user sign-out by navigating back to the root route (`/`). This action is triggered when the user successfully signs out from the profile screen.

2. **mfaAction**: A predefined action (defined earlier in the file at lines 88-99) that handles Multi-Factor Authentication (MFA) requirements. When MFA is required, it starts the MFA verification process and then navigates to the profile screen.

### actionCodeSettings

References the global `actionCodeSettings` configuration (defined at lines 23-29) that contains settings for email-based actions like password reset and email verification. This includes:

- URL for handling action codes
- Android minimum version requirements
- Package names for Android and iOS

### showMFATile

A boolean that controls whether the MFA (Multi-Factor Authentication) tile is displayed in the profile screen. The logic shows MFA options on:

- **Web platforms** (`kIsWeb`): Always shown for web applications
- **iOS** (`platform == TargetPlatform.iOS`): Native iOS MFA support
- **Android** (`platform == TargetPlatform.android`): Native Android MFA support

MFA is not shown on other platforms like desktop applications.

### showUnlinkConfirmationDialog

When set to `true`, displays a confirmation dialog when users attempt to unlink authentication providers (like Google, Facebook, etc.) from their account. This prevents accidental removal of authentication methods.

### showDeleteConfirmationDialog

When set to `true`, shows a confirmation dialog when users attempt to delete their account. This critical safety feature prevents accidental account deletion.

## Platform Detection

The configuration uses Flutter's platform detection to customize the UI based on the target platform:

```dart 267:267:lib/main.dart
final platform = Theme.of(context).platform;
```

This allows the app to provide different experiences across platforms while maintaining a consistent authentication flow.

## Security Considerations

This ProfileScreen configuration implements several security best practices:

1. **Account Deletion Protection**: Requires explicit confirmation before account deletion
2. **Provider Unlinking Safety**: Confirms before removing authentication providers
3. **MFA Platform Awareness**: Only shows MFA options on supported platforms
4. **Proper Navigation**: Uses `pushReplacementNamed` to prevent back navigation to sensitive screens after sign-out

## User Experience Features

The configuration provides a polished user experience through:

- **Clear Navigation**: Users are properly redirected after sign-out
- **MFA Integration**: Seamless handling of multi-factor authentication requirements
- **Platform Optimization**: UI adapts to platform capabilities
- **Confirmation Dialogs**: Prevents accidental destructive actions
- **Consistent Branding**: Uses the same action code settings as other auth screens

This ProfileScreen setup serves as a comprehensive example of how to configure Firebase UI Auth's profile management features with proper security, platform awareness, and user experience considerations.
