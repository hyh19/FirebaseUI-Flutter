# main.dart - App Tracking Transparency

## Overview

This document explains the `AppTrackingTransparencyCard` widget, which provides iOS users with a toggle to manage app tracking permissions as required by Apple's App Tracking Transparency framework.

## AppTrackingTransparencyCard Class

This stateful widget manages the display and interaction for app tracking permission status:

```dart 298:304:lib/main.dart
class AppTrackingTransparencyCard extends StatefulWidget {
  const AppTrackingTransparencyCard({super.key});

  @override
  State<AppTrackingTransparencyCard> createState() =>
      _AppTrackingTransparencyCardState();
}
```

### State Management

The widget uses a private state class to manage tracking permission status:

```dart 306:314:lib/main.dart
class _AppTrackingTransparencyCardState
    extends State<AppTrackingTransparencyCard> {
  bool _isAllowed = false;

  @override
  void initState() {
    super.initState();
    _checkTrackingStatus();
  }
```

## Permission Status Checking

The `_checkTrackingStatus` method queries the current tracking authorization status:

```dart 316:325:lib/main.dart
  Future<void> _checkTrackingStatus() async {
    try {
      final status = await AppTrackingTransparency.trackingAuthorizationStatus;
      setState(() {
        _isAllowed = status == TrackingStatus.authorized;
      });
    } catch (e) {
      // Handle error silently
    }
  }
```

### Error Handling

- **Silent Failure**: Errors are handled silently to prevent app crashes
- **Graceful Degradation**: Widget continues to function even if permission check fails
- **User Experience**: No error messages shown to avoid confusing users

## Permission Toggle Logic

The `_onToggleChanged` method handles user interaction with the tracking toggle:

```dart 327:367:lib/main.dart
  Future<void> _onToggleChanged(bool value) async {
    if (value && !_isAllowed) {
      // Request permission when toggling on
      try {
        final status =
            await AppTrackingTransparency.requestTrackingAuthorization();
        if (mounted) {
          setState(() {
            _isAllowed = status == TrackingStatus.authorized;
          });

          if (status != TrackingStatus.authorized) {
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(
                content: Text(
                  'Tracking permission denied. Enable in Settings > Privacy & Security > Tracking',
                ),
                duration: Duration(seconds: 4),
              ),
            );
          }
        }
      } catch (e) {
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Error requesting permission: $e')),
          );
        }
      }
    } else if (!value && _isAllowed) {
      // Can't turn off programmatically - show message
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text(
            'To disable tracking, go to Settings > Privacy & Security > Tracking',
          ),
          duration: Duration(seconds: 3),
        ),
      );
    }
  }
```

### Toggle Behavior

1. **Enabling Tracking**:
   - Shows system permission dialog
   - Updates UI based on user response
   - Shows snackbar if permission denied with instructions

2. **Disabling Tracking**:
   - Cannot be disabled programmatically (iOS limitation)
   - Shows instructional message directing users to Settings

### Error Handling

- **Permission Request Errors**: Shows error message via snackbar
- **Mounted Check**: Ensures widget is still in tree before updating state
- **User Feedback**: Provides clear instructions for manual permission management

## Widget UI

The widget renders a simple toggle interface:

```dart 369:386:lib/main.dart
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Text('App tracking allowed'),
          const SizedBox(width: 12),
          Switch(
            value: _isAllowed,
            onChanged: _onToggleChanged,
          ),
        ],
      ),
    );
  }
```

### UI Components

- **Padding**: Vertical padding for visual spacing
- **Row Layout**: Horizontal arrangement with centered alignment
- **Text Label**: Descriptive text for the toggle
- **Spacing**: SizedBox for visual separation
- **Switch Widget**: Material Design switch component

## iOS App Tracking Transparency Requirements

This widget addresses Apple's App Tracking Transparency requirements:

### When Required

- **iOS 14.5+**: Required for apps that track users across apps/websites
- **Data Collection**: Must be shown before collecting tracking data
- **User Consent**: Users must explicitly grant permission

### Integration Points

The widget is conditionally shown only on iOS platforms in the sign-in screen footer:

```dart 176:178:lib/main.dart
if (platform == TargetPlatform.iOS)
  const AppTrackingTransparencyCard(),
```

### Best Practices

- **Early Display**: Show permission request at app launch or first relevant screen
- **Clear Purpose**: Explain why tracking permission is needed
- **Fallback Behavior**: App should function without tracking permission
- **Settings Access**: Provide clear instructions for changing permission later

## State Management Considerations

### Lifecycle Management

- **initState**: Checks permission status when widget is created
- **mounted Check**: Prevents state updates on disposed widgets
- **Error Resilience**: Continues functioning even if permission API fails

### User Experience

- **Immediate Feedback**: UI updates immediately after permission changes
- **Clear Communication**: Snackbars provide clear next steps
- **Non-Blocking**: Permission request doesn't prevent app usage

## Platform-Specific Behavior

### iOS Implementation

- **System Dialog**: Uses native iOS permission dialog
- **Persistent Status**: Permission status persists across app sessions
- **Settings Integration**: Integrates with iOS Settings app

### Other Platforms

- **Conditional Display**: Only shown on iOS (TargetPlatform.iOS check)
- **No-Op on Other Platforms**: Safe to include in cross-platform code

## Privacy Compliance

This implementation helps apps comply with privacy regulations:

- **User Control**: Gives users control over tracking permissions
- **Transparency**: Clearly indicates tracking status
- **Consent Management**: Handles consent request and status tracking
- **Audit Trail**: Permission status is queryable for compliance reporting
