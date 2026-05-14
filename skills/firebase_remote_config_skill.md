# Firebase Remote Config Service Skill

Complete implementation guide for Firebase Remote Config in Flutter clean architecture projects.

---

## Package

```yaml
dependencies:
  firebase_remote_config: ^5.x.x
```

---

## File Structure

```
lib/core/services/remote_config/
└── firebase_remote_config_service.dart
```

---

## Complete Implementation

```dart
// lib/core/services/remote_config/firebase_remote_config_service.dart

import 'dart:convert';
import 'package:firebase_remote_config/firebase_remote_config.dart';
import 'package:flutter/foundation.dart';

class FirebaseRemoteConfigService {
  FirebaseRemoteConfigService._();

  static final FirebaseRemoteConfigService _instance =
      FirebaseRemoteConfigService._();

  factory FirebaseRemoteConfigService() => _instance;

  late final FirebaseRemoteConfig _remoteConfig;

  Future<void> init() async {
    _remoteConfig = FirebaseRemoteConfig.instance;

    await _remoteConfig.setConfigSettings(
      RemoteConfigSettings(
        fetchTimeout: const Duration(seconds: 10),
        minimumFetchInterval: Duration.zero, // use Duration(hours: 1) in prod
      ),
    );

    // Set defaults — always define fallback values here
    await _remoteConfig.setDefaults({
      'base_url': 'https://your-default-api.com',
      'socket_url': 'https://your-default-socket.com',
      'remoteConfigData': '{"feature_x":false,"version":"1.0.0"}',
    });

    try {
      final updated = await _remoteConfig.fetchAndActivate();
      debugPrint('Remote Config updated: $updated');
    } catch (e) {
      debugPrint('Failed to fetch remote config: $e');
      // App continues with default values — never crash here
    }
  }

  // ─── Basic String Values ───────────────────────────────────────────────────

  String get baseUrl => _remoteConfig.getString('base_url');
  String get socketUrl => _remoteConfig.getString('socket_url');

  // ─── JSON Config Model ─────────────────────────────────────────────────────

  Map<String, dynamic> get _remoteJson {
    try {
      final jsonString = _remoteConfig.getString('remoteConfigData');
      if (jsonString.isEmpty) return {};
      return jsonDecode(jsonString) as Map<String, dynamic>;
    } catch (e) {
      debugPrint('Remote Config JSON parse error: $e');
      return {};
    }
  }

  RemoteConfigModel get configModel {
    try {
      return RemoteConfigModel.fromJson(_remoteJson);
    } catch (e) {
      debugPrint('Remote Config model error: $e');
      return RemoteConfigModel.empty();
    }
  }

  // ─── Direct Feature Flag Getters ──────────────────────────────────────────

  bool get featureX => configModel.featureX;
  String get appVersion => configModel.version;
}

// ─── Model ────────────────────────────────────────────────────────────────────

class RemoteConfigModel {
  final bool featureX;
  final String version;

  const RemoteConfigModel({
    required this.featureX,
    required this.version,
  });

  factory RemoteConfigModel.fromJson(Map<String, dynamic> json) {
    return RemoteConfigModel(
      featureX: json['feature_x'] ?? false,
      version: json['version'] ?? '1.0.0',
    );
  }

  factory RemoteConfigModel.empty() {
    return const RemoteConfigModel(
      featureX: false,
      version: '1.0.0',
    );
  }
}
```

---

## DI Registration

Remote Config is a singleton by itself (factory pattern inside). Initialize it in `main_common.dart`, NOT in service locator:

```dart
// lib/main_common.dart
Future<void> _initializeServices() async {
  try {
    final remoteConfig = FirebaseRemoteConfigService();
    await remoteConfig.init();
  } catch (e) {
    debugPrint('Remote Config init failed: $e');
  }
}
```

To use it anywhere without DI:

```dart
final remoteConfig = FirebaseRemoteConfigService(); // returns same singleton
final url = remoteConfig.baseUrl;
```

If you need it in Cubit/Repository via DI, register as lazy singleton:

```dart
// lib/core/di/service_locator.dart
sl.registerLazySingleton<FirebaseRemoteConfigService>(
  () => FirebaseRemoteConfigService(),
);
```

---

## Usage Examples

### Feature Flag Check

```dart
final config = FirebaseRemoteConfigService();

if (config.featureX) {
  // show feature
}
```

### In DataSource

```dart
class AvatarRemoteDataSourceImpl implements AvatarRemoteDataSource {
  final FirebaseRemoteConfigService _remoteConfig;

  AvatarRemoteDataSourceImpl(this._remoteConfig);

  @override
  Future<void> generateAvatar() async {
    if (!_remoteConfig.featureX) {
      throw FeatureDisabledException('Avatar generation is disabled');
    }
    // proceed
  }
}
```

### In Socket Client

```dart
class ChatSocketClient {
  final FirebaseRemoteConfigService _remoteConfig;

  ChatSocketClient(this._remoteConfig);

  void connect() {
    final url = _remoteConfig.socketUrl;
    // connect to socket
  }
}
```

---

## Adding New Remote Config Keys

**Step 1** — Add default value in `setDefaults`:

```dart
await _remoteConfig.setDefaults({
  'new_feature_enabled': 'false',
});
```

**Step 2** — Add getter:

```dart
bool get newFeatureEnabled =>
    _remoteConfig.getString('new_feature_enabled') == 'true';
```

**Step 3** — If it's part of JSON config, add to `RemoteConfigModel`:

```dart
factory RemoteConfigModel.fromJson(Map<String, dynamic> json) {
  return RemoteConfigModel(
    newFeature: json['new_feature'] ?? false,
    // ...
  );
}
```

**Step 4** — Add the key in Firebase Console → Remote Config.

---

## Rules

- Always define defaults — app must work without network
- Never crash if fetch fails — use try/catch and fallback to defaults
- `minimumFetchInterval: Duration.zero` in dev, `Duration(hours: 1)` in prod
- Use JSON key `remoteConfigData` for complex config — one key, one model
- Do NOT call `fetchAndActivate()` multiple times — only once on init
