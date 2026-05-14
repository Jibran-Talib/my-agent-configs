# Notification Service Skill

Complete implementation guide for FCM + Local Notifications in Flutter projects.

---

## Packages

```yaml
dependencies:
  firebase_messaging: ^15.x.x
  flutter_local_notifications: ^18.x.x
  firebase_core: ^3.x.x
```

---

## File Structure

```
lib/core/services/notification_services/
└── notification_service.dart
```

---

## Complete Implementation

```dart
// lib/core/services/notification_services/notification_service.dart

import 'dart:async';
import 'dart:convert';
import 'dart:developer';
import 'dart:io';

import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

import '../../../firebase_options.dart';
import '../storage_service.dart';

class NotificationService {
  NotificationService(this._storageService);

  static const String _channelId = 'default_channel';
  static const String _channelName = 'Default Channel';
  static const String _channelDesc = 'Used for general push notifications.';

  final FirebaseMessaging _messaging = FirebaseMessaging.instance;
  final StorageService _storageService;
  final FlutterLocalNotificationsPlugin _localNotifications =
      FlutterLocalNotificationsPlugin();

  StreamSubscription<String>? _tokenRefreshSub;
  String? _cachedToken;
  bool _isInitialized = false;

  // ─── Background Handler (top-level function) ───────────────────────────────

  @pragma('vm:entry-point')
  static Future<void> firebaseMessagingBackgroundHandler(
    RemoteMessage message,
  ) async {
    await Firebase.initializeApp(
      options: DefaultFirebaseOptions.currentPlatform,
    );
    log('Background Message: ${message.messageId}');
  }

  // ─── Initialize ───────────────────────────────────────────────────────────

  Future<void> initialize() async {
    if (_isInitialized) return;

    FirebaseMessaging.onBackgroundMessage(firebaseMessagingBackgroundHandler);

    await _initLocalNotifications();
    await _requestPermissions();
    await _configureForeground();

    if (Platform.isAndroid) {
      await _createAndroidChannel();
    }

    // Token refresh listener
    _tokenRefreshSub = _messaging.onTokenRefresh.listen((token) async {
      await _saveToken(token, source: 'refresh');
    });

    // Get and cache FCM token
    final token = await getFcmToken(forceRefresh: true);
    log('FCM Token: $token');

    // Foreground messages — show local notification on Android
    FirebaseMessaging.onMessage.listen((message) async {
      log('Foreground Message: ${message.data}');
      if (Platform.isAndroid) {
        await _showLocalNotification(message);
      }
    });

    // App opened from terminated state
    final initialMessage = await _messaging.getInitialMessage();
    if (initialMessage != null) {
      _handleNotificationTap(initialMessage.data);
    }

    // App opened from background
    FirebaseMessaging.onMessageOpenedApp.listen((message) {
      _handleNotificationTap(message.data);
    });

    _isInitialized = true;
  }

  // ─── Local Notifications Setup ────────────────────────────────────────────

  Future<void> _initLocalNotifications() async {
    const androidInit = AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosInit = DarwinInitializationSettings(
      requestAlertPermission: false,
      requestBadgePermission: false,
      requestSoundPermission: false,
    );

    await _localNotifications.initialize(
      const InitializationSettings(android: androidInit, iOS: iosInit),
      onDidReceiveNotificationResponse: (response) {
        try {
          final data = jsonDecode(response.payload ?? '{}') as Map<String, dynamic>;
          _handleNotificationTap(data);
        } catch (e) {
          debugPrint('Local tap error: $e');
        }
      },
    );
  }

  Future<void> _requestPermissions() async {
    await _messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
    );

    if (Platform.isAndroid) {
      await _localNotifications
          .resolvePlatformSpecificImplementation<
              AndroidFlutterLocalNotificationsPlugin>()
          ?.requestNotificationsPermission();
    }

    if (Platform.isIOS) {
      await _localNotifications
          .resolvePlatformSpecificImplementation<
              IOSFlutterLocalNotificationsPlugin>()
          ?.requestPermissions(alert: true, badge: true, sound: true);
    }
  }

  Future<void> _configureForeground() async {
    await _messaging.setForegroundNotificationPresentationOptions(
      alert: true,
      badge: true,
      sound: true,
    );
  }

  Future<void> _createAndroidChannel() async {
    const channel = AndroidNotificationChannel(
      _channelId,
      _channelName,
      description: _channelDesc,
      importance: Importance.max,
      playSound: true,
    );

    await _localNotifications
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(channel);
  }

  // ─── Show Local Notification ──────────────────────────────────────────────

  Future<void> _showLocalNotification(RemoteMessage message) async {
    const details = NotificationDetails(
      android: AndroidNotificationDetails(
        _channelId,
        _channelName,
        channelDescription: _channelDesc,
        importance: Importance.max,
        priority: Priority.high,
        playSound: true,
      ),
      iOS: DarwinNotificationDetails(
        presentAlert: true,
        presentBadge: true,
        presentSound: true,
      ),
    );

    await _localNotifications.show(
      message.messageId?.hashCode ?? DateTime.now().millisecondsSinceEpoch,
      message.notification?.title ?? message.data['title'] ?? 'Notification',
      message.notification?.body ?? message.data['body'] ?? '',
      details,
      payload: jsonEncode(message.data),
    );
  }

  // ─── FCM Token ────────────────────────────────────────────────────────────

  Future<String?> getFcmToken({bool forceRefresh = false}) async {
    if (!forceRefresh && _cachedToken != null) return _cachedToken;

    final stored = _storageService.getFcmToken();
    if (!forceRefresh && stored != null && stored.isNotEmpty) {
      _cachedToken = stored;
      return stored;
    }

    try {
      if (Platform.isIOS) {
        await _messaging.getAPNSToken();
      }
      final token = await _messaging.getToken();
      if (token != null) {
        await _saveToken(token, source: 'getToken');
        return token;
      }
    } catch (e) {
      debugPrint('Token error: $e');
    }

    return null;
  }

  Future<void> _saveToken(String token, {required String source}) async {
    final clean = token.trim();
    if (clean.isEmpty) return;
    _cachedToken = clean;
    await _storageService.saveFcmToken(clean);
    log('FCM Token saved from $source');
  }

  // ─── Handle Notification Tap ──────────────────────────────────────────────
  // Override this method per-project to handle navigation on tap

  void _handleNotificationTap(Map<String, dynamic> data) {
    log('Notification tapped: $data');

    final type = data['type']?.toString();

    switch (type) {
      case 'chat':
        final id = data['subId']?.toString();
        if (id != null && id.isNotEmpty) {
          // Navigate to chat room
          // ChatNavigation.toChatRoom(context, id);
        }
        break;

      case 'friend_request':
        // Navigate to friend requests screen
        break;

      default:
        log('Unknown notification type: $type');
    }
  }

  // ─── Helpers ──────────────────────────────────────────────────────────────

  bool _parseBool(dynamic value) {
    if (value is bool) return value;
    if (value is String) return value.toLowerCase() == 'true';
    return false;
  }

  int _parseInt(dynamic value) {
    if (value is int) return value;
    if (value is String) return int.tryParse(value) ?? 0;
    return 0;
  }

  Future<void> dispose() async {
    await _tokenRefreshSub?.cancel();
    _tokenRefreshSub = null;
    _isInitialized = false;
  }
}
```

---

## DI Registration

```dart
// lib/core/di/service_locator.dart
sl.registerLazySingleton<NotificationService>(
  () => NotificationService(sl<StorageService>()),
);
```

---

## Initialization

```dart
// lib/main_common.dart
Future<void> _initializeServices() async {
  try {
    await sl<NotificationService>().initialize();
  } catch (e) {
    debugPrint('NotificationService init failed: $e');
  }
}
```

---

## Usage — Sync FCM Token After Login

```dart
// In AuthCubit after successful login
Future<void> _syncFcmToken() async {
  try {
    String? token = await _notificationService.getFcmToken();

    if (token == null || token.isEmpty) {
      token = await _notificationService.getFcmToken(forceRefresh: true);
    }

    if (token == null || token.isEmpty) return;

    await _updateFcmUseCase(UpdateFcmParams(deviceToken: token));
  } catch (e) {
    log('FCM sync error: $e');
  }
}
```

---

## Adding New Notification Types

Add a new case in `_handleNotificationTap`:

```dart
case 'order_update':
  final orderId = data['orderId']?.toString();
  if (orderId != null) {
    // Navigate to order detail
  }
  break;
```

---

## Android Setup (AndroidManifest.xml)

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

<application>
  <!-- Default notification channel -->
  <meta-data
    android:name="com.google.firebase.messaging.default_notification_channel_id"
    android:value="default_channel"/>
</application>
```

---

## Rules

- Always initialize in `main_common.dart`, not in DI
- Background handler must be a top-level function (not inside class)
- On Android: always show local notification for foreground messages (iOS handles it automatically)
- Always cache FCM token in `StorageService` — avoid repeated fetches
- Guard with `_isInitialized` to prevent double initialization
- Never call `getFcmToken()` before `initialize()` completes
- Handle all notification types in `_handleNotificationTap` — add cases as features grow
