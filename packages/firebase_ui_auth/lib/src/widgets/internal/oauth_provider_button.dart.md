# OAuth Provider Button Implementation

This file implements the OAuth provider button widget for Firebase UI Auth, providing a unified interface for OAuth-based authentication across different providers like Google, Facebook, Twitter, and Apple.

## Overview

The `OAuthProviderButton` class creates interactive buttons that handle OAuth authentication flows. It integrates with Firebase UI OAuth providers and provides consistent theming, error handling, and loading states.

## Key Components

### OAuthButtonVariant Enum

```dart 17:21:packages/firebase_ui_auth/lib/src/widgets/internal/oauth_provider_button.dart
enum OAuthButtonVariant {
  // ignore: constant_identifier_names
  icon_and_text,
  icon,
}
```

Defines two display variants for OAuth buttons:

- `icon_and_text`: Shows both provider icon and text label
- `icon`: Shows only the provider icon

### Error Listener Widget

```dart 23:38:packages/firebase_ui_auth/lib/src/widgets/internal/oauth_provider_button.dart
class _ErrorListener extends StatelessWidget {
  const _ErrorListener();

  @override
  Widget build(BuildContext context) {
    final state = AuthState.of(context);
    if (state is AuthFailed) {
      return Padding(
        padding: const EdgeInsets.symmetric(vertical: 4),
        child: ErrorText(exception: state.exception),
      );
    }

    return const SizedBox.shrink();
  }
}
```

A private widget that listens for authentication failures and displays error messages below the button. It uses the `AuthState.of(context)` to access the current authentication state and shows an `ErrorText` widget when authentication fails.

### OAuthProviderButton Class

The main widget class that implements the OAuth provider button functionality.

#### Constructor Parameters

```dart 43:93:packages/firebase_ui_auth/lib/src/widgets/internal/oauth_provider_button.dart
class OAuthProviderButton extends StatelessWidget {
  /// {@template ui.auth.widgets.oauth_provider_button.provider}
  /// An instance of [ffui_oauth.OAuthProvider] that should be used to
  /// authenticate.
  /// {@endtemplate}
  final ffui_oauth.OAuthProvider provider;

  /// {@macro ui.auth.auth_action}
  final AuthAction? action;

  /// {@macro ui.auth.auth_controller.auth}
  final fba.FirebaseAuth? auth;

  /// {@macro ui.auth.widgets.oauth_provider_button.oauth_button_variant}
  final OAuthButtonVariant? variant;

  /// {@macro ui.auth.widgets.oauth_provider_button}
  const OAuthProviderButton({
    super.key,

    /// {@macro ui.auth.widgets.oauth_provider_button.provider}
    required this.provider,

    /// {@macro ui.auth.widgets.oauth_provider_button.oauth_button_variant}
    this.variant = OAuthButtonVariant.icon_and_text,

    /// {@macro ui.auth.auth_action}
    this.action,

    /// {@macro ui.auth.auth_controller.auth}
    this.auth,
  });
```

- `provider`: Required OAuth provider instance from `firebase_ui_oauth`
- `action`: Optional authentication action (sign in/sign up)
- `auth`: Optional Firebase Auth instance
- `variant`: Button display variant (defaults to icon and text)

#### Provider Label Resolution

```dart 60:76:packages/firebase_ui_auth/lib/src/widgets/internal/oauth_provider_button.dart
  /// Returns a text that should be displayed on the button.
  static String resolveProviderButtonLabel(
    String providerId,
    FirebaseUILocalizationLabels labels,
  ) {
    switch (providerId) {
      case 'google.com':
        return labels.signInWithGoogleButtonText;
      case 'facebook.com':
        return labels.signInWithFacebookButtonText;
      case 'twitter.com':
        return labels.signInWithTwitterButtonText;
      case 'apple.com':
        return labels.signInWithAppleButtonText;
      default:
        throw Exception('Unknown providerId $providerId');
    }
  }
```

Static method that maps provider IDs to localized button labels. It supports the four main OAuth providers and throws an exception for unknown providers.

#### Widget Build Method

```dart 95:135:packages/firebase_ui_auth/lib/src/widgets/internal/oauth_provider_button.dart
  @override
  Widget build(BuildContext context) {
    final labels = FirebaseUILocalizations.labelsOf(context);
    final brightness = Theme.of(context).brightness;

    return AuthFlowBuilder<OAuthController>(
      provider: provider,
      action: action,
      auth: auth,
      builder: (context, state, ctrl, child) {
        final button = ffui_oauth.OAuthProviderButtonBase(
          provider: provider,
          action: action,
          isLoading: state is SigningIn || state is CredentialReceived,
          onTap: () => ctrl.signIn(Theme.of(context).platform),
          overrideDefaultTapAction: true,
          loadingIndicator: LoadingIndicator(
            size: 19,
            borderWidth: 1,
            color: provider.style.color.getValue(brightness),
          ),
          label: variant == OAuthButtonVariant.icon
              ? ''
              : provider.style.label ??
                  resolveProviderButtonLabel(provider.providerId, labels),
          auth: auth,
        );

        if (variant == OAuthButtonVariant.icon) {
          return button;
        }

        return Column(
          children: [
            button,
            const _ErrorListener(),
          ],
        );
      },
    );
  }
```

The build method creates an `AuthFlowBuilder` that manages the OAuth authentication flow:

1. **State Management**: Uses `AuthFlowBuilder<OAuthController>` to handle OAuth authentication state
2. **Button Creation**: Creates an `OAuthProviderButtonBase` from the OAuth package
3. **Loading States**: Shows loading indicator during `SigningIn` or `CredentialReceived` states
4. **Tap Handling**: Calls `ctrl.signIn()` with the current platform when tapped
5. **Label Logic**: Uses provider's custom label or falls back to localized labels
6. **Layout**: For icon-only variant, returns just the button; for icon-and-text, wraps in a Column with error listener

## Integration Points

### Firebase UI OAuth

The widget depends on `firebase_ui_oauth.OAuthProvider` instances and `OAuthProviderButtonBase` for the actual button implementation.

### Localization

Uses `FirebaseUILocalizations` to provide localized button text for different languages.

### Authentication Flow

Integrates with the broader Firebase UI Auth system through `AuthFlowBuilder` and authentication controllers.

### Theming

Respects the current theme brightness for loading indicator colors and uses provider-specific styling.

## Usage Example

```dart
// Google sign-in button with icon and text
OAuthProviderButton(
  provider: GoogleProvider(clientId: 'your-client-id'),
  action: AuthAction.signIn,
)

// Facebook icon-only button
OAuthProviderButton(
  provider: FacebookProvider(clientId: 'your-client-id'),
  variant: OAuthButtonVariant.icon,
)
```

## Error Handling

The widget automatically displays authentication errors below the button when they occur, providing immediate feedback to users about sign-in failures.
