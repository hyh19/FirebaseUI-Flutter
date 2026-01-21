# main.dart - Localization Setup

## Overview

This document explains the localization configuration in the Firebase UI Auth example app, including custom label overrides and Material Design localization delegates.

## Label Overrides Class

The app provides custom localization overrides for specific UI elements:

```dart 56:64:lib/main.dart
// Overrides a label for en locale
// To add localization for a custom language follow the guide here:
// https://flutter.dev/docs/development/accessibility-and-localization/internationalization#an-alternative-class-for-the-apps-localized-resources
class LabelOverrides extends DefaultLocalizations {
  const LabelOverrides();

  @override
  String get emailInputLabel => 'Enter your email';
}
```

### Key Features

- **Extends DefaultLocalizations**: Inherits Firebase UI's default English localizations
- **Custom Email Label**: Overrides the default email input field label
- **Constructor**: Uses const constructor for performance optimization
- **Documentation**: Includes helpful comments about adding custom language support

### Usage in MaterialApp

The LabelOverrides class is integrated into the app's localization system:

```dart 287:293:lib/main.dart
      supportedLocales: const [Locale('en')],
      localizationsDelegates: [
        FirebaseUILocalizations.withDefaultOverrides(const LabelOverrides()),
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        FirebaseUILocalizations.delegate,
      ],
```

## Localization Configuration

The app configures multiple localization delegates for comprehensive internationalization support:

### Supported Locales

- **English Only**: Currently supports only English locale (`Locale('en')`)

### Localization Delegates

1. **FirebaseUILocalizations**: Firebase UI's localization delegate with custom overrides
2. **GlobalMaterialLocalizations**: Flutter's Material Design localizations
3. **GlobalWidgetsLocalizations**: Flutter's basic widget localizations
4. **FirebaseUILocalizations**: Base Firebase UI localizations

### Custom Override Integration

The `FirebaseUILocalizations.withDefaultOverrides()` method merges custom overrides with default Firebase UI strings:

- **Default Behavior**: Uses Firebase UI's built-in English strings
- **Override Mechanism**: Replaces specific keys with custom values
- **Fallback**: Falls back to default strings for non-overridden keys

## Extending to Multiple Languages

To add support for additional languages, follow these steps:

1. **Create Language-Specific Classes**: Extend `DefaultLocalizations` for each language
2. **Override Required Methods**: Implement all necessary localization methods
3. **Add to Supported Locales**: Include new locales in `supportedLocales`
4. **Register Delegates**: Add language-specific delegates to `localizationsDelegates`

### Example Structure for Multiple Languages

```dart
// For Spanish support
class SpanishOverrides extends DefaultLocalizations {
  const SpanishOverrides();

  @override
  String get emailInputLabel => 'Ingrese su correo electrónico';
  // ... other overrides
}

// In MaterialApp
supportedLocales: const [
  Locale('en'),
  Locale('es'),
],
localizationsDelegates: [
  FirebaseUILocalizations.withDefaultOverrides(const LabelOverrides()),
  FirebaseUILocalizations.withDefaultOverrides(const SpanishOverrides()),
  // ... other delegates
],
```

## Best Practices

- **Minimal Overrides**: Only override labels that need customization
- **Consistent Terminology**: Maintain consistent terminology across the app
- **Accessibility**: Ensure custom labels work well with screen readers
- **Testing**: Test localization in both LTR and RTL languages
