# LoginView Implementation

## Overview

The `LoginView` class is a core component of the Firebase UI Auth package that provides a flexible, customizable authentication view for Flutter applications. It serves as the foundation for building custom sign-in and registration screens, supporting multiple authentication providers including email/password, phone verification, OAuth providers, and email link authentication.

## Key Components

### AuthViewContentBuilder Typedef

```dart 16:19:packages/firebase_ui_auth/lib/src/views/login_view.dart
typedef AuthViewContentBuilder = Widget Function(
  BuildContext context,
  AuthAction action,
);
```

This typedef defines a function signature for building custom content within the login view. It allows developers to inject custom widgets that can respond to the current authentication action (sign-in or sign-up).

### LoginView Class

The `LoginView` is a stateful widget that encapsulates the authentication UI logic:

```dart 25:79:packages/firebase_ui_auth/lib/src/views/login_view.dart
class LoginView extends StatefulWidget {
  /// {@macro ui.auth.auth_controller.auth}
  final fba.FirebaseAuth? auth;

  /// {@macro ui.auth.auth_action}
  final AuthAction action;

  /// Indicates whether icon-only or icon and text OAuth buttons should be used.
  /// Icon-only buttons are placed in a row.
  final OAuthButtonVariant? oauthButtonVariant;
  final bool? showTitle;
  final String? email;

  /// Whether the "Login/Register" link should be displayed. The link changes
  /// the type of the [AuthAction] from [AuthAction.signIn]
  /// and [AuthAction.signUp] and vice versa.
  final bool? showAuthActionSwitch;

  /// {@template ui.auth.views.login_view.footer_builder}
  /// A returned widget would be placed down the authentication related widgets.
  /// {@endtemplate}
  final AuthViewContentBuilder? footerBuilder;

  /// {@template ui.auth.views.login_view.subtitle_builder}
  /// A returned widget would be placed up the authentication related widgets.
  /// {@endtemplate}
  final AuthViewContentBuilder? subtitleBuilder;

  final List<AuthProvider> providers;

  /// A label that would be used for the "Sign in" button.
  final String? actionButtonLabelOverride;

  /// {@macro ui.auth.widgets.email_from.showPasswordVisibilityToggle}
  final bool showPasswordVisibilityToggle;

  /// {@macro ui.auth.views.login_view}
  const LoginView({
    super.key,
    required this.action,
    required this.providers,
    this.oauthButtonVariant = OAuthButtonVariant.icon_and_text,
    this.auth,
    this.showTitle = true,
    this.email,
    this.showAuthActionSwitch,
    this.footerBuilder,
    this.subtitleBuilder,
    this.actionButtonLabelOverride,
    this.showPasswordVisibilityToggle = false,
  });
```

#### Constructor Parameters

- **`auth`**: Optional Firebase Auth instance for custom authentication handling
- **`action`**: Required `AuthAction` (sign-in or sign-up) that determines the view's behavior
- **`oauthButtonVariant`**: Controls OAuth button appearance (icon-only vs icon-and-text)
- **`showTitle`**: Whether to display the authentication title
- **`email`**: Pre-filled email address for email authentication
- **`showAuthActionSwitch`**: Whether to show the toggle between sign-in and sign-up modes
- **`footerBuilder`** & **`subtitleBuilder`**: Custom content builders for extending the UI
- **`providers`**: List of authentication providers to display
- **`actionButtonLabelOverride`**: Custom label for the action button
- **`showPasswordVisibilityToggle`**: Controls password visibility toggle in email forms

## State Management

### _LoginViewState Class

The state class manages the view's internal state and handles UI rendering:

```dart 81:91:packages/firebase_ui_auth/lib/src/views/login_view.dart
class _LoginViewState extends State<LoginView> {
  late AuthAction _action = widget.action;
  bool get _showTitle => widget.showTitle ?? true;
  bool get _showAuthActionSwitch => widget.showAuthActionSwitch ?? true;
  bool _buttonsBuilt = false;

  void setAction(AuthAction action) {
    setState(() {
      _action = action;
    });
  }
```

Key state variables:

- **`_action`**: Current authentication action (sign-in/sign-up)
- **`_buttonsBuilt`**: Flag to prevent duplicate OAuth button rendering
- **`setAction`**: Method to programmatically change the authentication action

## OAuth Button Rendering

### _buildOAuthButtons Method

This method handles the creation and layout of OAuth provider buttons:

```dart 93:121:packages/firebase_ui_auth/lib/src/views/login_view.dart
  Widget _buildOAuthButtons(TargetPlatform platform) {
    final oauthProviders = widget.providers
        .whereType<OAuthProvider>()
        .where((element) => element.supportsPlatform(platform));

    _buttonsBuilt = true;

    final oauthButtonsList = oauthProviders.map((provider) {
      return OAuthProviderButton(
        provider: provider,
        auth: widget.auth,
        action: _action,
        variant: widget.oauthButtonVariant,
      );
    }).toList();

    if (widget.oauthButtonVariant == OAuthButtonVariant.icon_and_text) {
      return Column(
        mainAxisSize: MainAxisSize.min,
        children: oauthButtonsList,
      );
    } else {
      return Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        mainAxisSize: MainAxisSize.min,
        children: oauthButtonsList,
      );
    }
  }
```

The method:

1. Filters OAuth providers that support the current platform
2. Sets the `_buttonsBuilt` flag to prevent duplicate rendering
3. Creates `OAuthProviderButton` widgets for each provider
4. Arranges buttons in a Column (for icon-and-text) or Row (for icon-only) layout

## Authentication Action Toggle

### _handleDifferentAuthAction Method

Handles switching between sign-in and sign-up modes:

```dart 123:133:packages/firebase_ui_auth/lib/src/views/login_view.dart
  void _handleDifferentAuthAction(BuildContext context) {
    if (_action == AuthAction.signIn) {
      setState(() {
        _action = AuthAction.signUp;
      });
    } else {
      setState(() {
        _action = AuthAction.signIn;
      });
    }
  }
```

This method toggles between `AuthAction.signIn` and `AuthAction.signUp`, updating the UI accordingly.

## Header Construction

### _buildHeader Method

Builds the title and action switch components:

```dart 135:197:packages/firebase_ui_auth/lib/src/views/login_view.dart
  List<Widget> _buildHeader(BuildContext context) {
    final l = FirebaseUILocalizations.labelsOf(context);

    late String title;
    late String hint;
    late String actionText;

    if (_action == AuthAction.signIn) {
      title = l.signInText;
      hint = l.registerHintText;
      actionText = l.registerText;
    } else if (_action == AuthAction.signUp) {
      title = l.registerText;
      hint = l.signInHintText;
      actionText = l.signInText;
    }

    final isCupertino = CupertinoUserInterfaceLevel.maybeOf(context) != null;
    TextStyle? hintStyle;
    late Color registerTextColor;

    if (isCupertino) {
      final theme = CupertinoTheme.of(context);
      registerTextColor = theme.primaryColor;
      hintStyle = theme.textTheme.textStyle.copyWith(fontSize: 12);
    } else {
      final theme = Theme.of(context);
      hintStyle = Theme.of(context).textTheme.bodySmall;
      registerTextColor = theme.colorScheme.primary;
    }

    return [
      Title(text: title),
      const SizedBox(height: 16),
      if (widget.subtitleBuilder != null)
        widget.subtitleBuilder!(
          context,
          _action,
        ),
      if (_showAuthActionSwitch) ...[
        Text.rich(
          TextSpan(
            children: [
              TextSpan(
                text: '$hint ',
                style: hintStyle,
              ),
              TextSpan(
                text: actionText,
                style: Theme.of(context).textTheme.labelLarge?.copyWith(
                      color: registerTextColor,
                    ),
                mouseCursor: SystemMouseCursors.click,
                recognizer: TapGestureRecognizer()
                  ..onTap = () => _handleDifferentAuthAction(context),
              ),
            ],
          ),
        ),
        const SizedBox(height: 16),
      ]
    ];
  }
```

This method:

1. Retrieves localized strings based on the current action
2. Adapts styling for both Material and Cupertino themes
3. Creates a clickable text span for switching between sign-in/sign-up
4. Includes optional subtitle content via the `subtitleBuilder`

## Lifecycle Management

### didUpdateWidget Method

Handles widget updates, particularly changes to the authentication action:

```dart 199:205:packages/firebase_ui_auth/lib/src/views/login_view.dart
  @override
  void didUpdateWidget(covariant LoginView oldWidget) {
    if (oldWidget.action != widget.action) {
      _action = widget.action;
    }
    super.didUpdateWidget(oldWidget);
  }
```

Ensures the internal `_action` state stays synchronized with the widget's `action` property.

## Main Build Method

### build Method

The core rendering logic that assembles all components:

```dart 207:256:packages/firebase_ui_auth/lib/src/views/login_view.dart
  @override
  Widget build(BuildContext context) {
    final l = FirebaseUILocalizations.labelsOf(context);
    final platform = Theme.of(context).platform;
    _buttonsBuilt = false;

    return IntrinsicHeight(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          if (_showTitle) ..._buildHeader(context),
          for (var provider in widget.providers)
            if (provider.supportsPlatform(platform))
              if (provider is EmailAuthProvider) ...[
                const SizedBox(height: 8),
                EmailForm(
                  key: ValueKey(_action),
                  auth: widget.auth,
                  action: _action,
                  provider: provider,
                  email: widget.email,
                  actionButtonLabelOverride: widget.actionButtonLabelOverride,
                  showPasswordVisibilityToggle:
                      widget.showPasswordVisibilityToggle,
                )
              ] else if (provider is PhoneAuthProvider) ...[
                const SizedBox(height: 8),
                PhoneVerificationButton(
                  label: l.signInWithPhoneButtonText,
                  action: _action,
                  auth: widget.auth,
                ),
                const SizedBox(height: 8),
              ] else if (provider is EmailLinkAuthProvider) ...[
                const SizedBox(height: 8),
                EmailLinkSignInButton(
                  auth: widget.auth,
                  provider: provider,
                ),
              ] else if (provider is OAuthProvider && !_buttonsBuilt)
                _buildOAuthButtons(platform),
          if (widget.footerBuilder != null)
            widget.footerBuilder!(
              context,
              _action,
            ),
        ],
      ),
    );
  }
```

The build method:

1. Resets the `_buttonsBuilt` flag for each rebuild
2. Uses `IntrinsicHeight` and `Column` for flexible vertical layout
3. Conditionally renders the header based on `_showTitle`
4. Iterates through providers, rendering appropriate UI components:
   - `EmailForm` for email authentication
   - `PhoneVerificationButton` for phone authentication
   - `EmailLinkSignInButton` for email link authentication
   - OAuth buttons for OAuth providers (only once due to `_buttonsBuilt` flag)
5. Includes optional footer content via `footerBuilder`

## Key Design Patterns

### Provider-Based Architecture

The view uses a provider-based approach where different authentication methods are encapsulated in `AuthProvider` subclasses. This allows for:

- Easy extensibility of new authentication methods
- Platform-specific provider filtering
- Consistent provider interface

### State Synchronization

The `_action` state is carefully managed to stay synchronized between the widget's properties and internal state, with proper handling in both `initState` and `didUpdateWidget`.

### Platform Awareness

The component is designed to work across different platforms:

- Filters providers based on platform support
- Adapts UI styling for Material and Cupertino themes
- Uses appropriate localization strings

### Flexible Customization

Through builder functions (`footerBuilder`, `subtitleBuilder`) and various configuration options, the view allows extensive customization while maintaining a consistent authentication flow.

## Usage Context

This `LoginView` is typically used within higher-level screens like `SignInScreen` or `RegisterScreen`, providing the core authentication UI that can be customized and extended as needed for specific application requirements.
