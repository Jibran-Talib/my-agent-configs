# Location Service Skill

Complete implementation guide for Location (Geolocator + Google Maps Geocoding) in Flutter projects.

---

## Packages

```yaml
dependencies:
  geolocator: ^13.x.x
  dio: ^5.x.x  # for geocoding API calls
```

---

## File Structure

```
lib/core/services/
└── location_service.dart

lib/shared/cubit/location/
└── location_cubit.dart
```

---

## Part 1 — Core Location Service (Geolocator)

```dart
// lib/core/services/location_service.dart

import 'dart:developer';
import 'package:geolocator/geolocator.dart';

class LocationService {
  Future<Position?> getCurrentPosition() async {
    try {
      final isEnabled = await Geolocator.isLocationServiceEnabled();
      if (!isEnabled) return null;

      var permission = await Geolocator.checkPermission();
      if (permission == LocationPermission.denied) {
        permission = await Geolocator.requestPermission();
      }

      if (permission == LocationPermission.denied ||
          permission == LocationPermission.deniedForever) {
        return null;
      }

      return await Geolocator.getCurrentPosition(
        locationSettings: const LocationSettings(
          accuracy: LocationAccuracy.high,
        ),
      );
    } catch (e, stackTrace) {
      log('LocationService error: $e');
      log('Stack: $stackTrace');
      return null;
    }
  }
}
```

---

## Part 2 — Location Cubit

```dart
// lib/shared/cubit/location/location_cubit.dart

import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:geolocator/geolocator.dart';
import '../../../core/services/location_service.dart';

class LocationCubit extends Cubit<Position?> {
  final LocationService _service;

  LocationCubit(this._service) : super(null);

  Future<void> fetchLocation() async {
    final position = await _service.getCurrentPosition();
    emit(position);
  }
}
```

---

## Part 3 — Geocoding Service (Google Maps API)

```dart
// lib/shared/widgets/location_component/location_service/location_services.dart

import 'dart:convert';
import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';

class GeocodingService {
  final Dio dio;
  final String placesApiKey;

  GeocodingService({required this.dio, required this.placesApiKey});

  // ─── Address → LatLng ────────────────────────────────────────────────────

  Future<AddressDetails> getLatLngFromAddress(String address) async {
    final url =
        'https://maps.googleapis.com/maps/api/geocode/json?address=${Uri.encodeComponent(address)}&key=$placesApiKey';

    try {
      final response = await dio.get(url);

      if (response.statusCode == 200) {
        final results = response.data['results'] as List<dynamic>;
        if (results.isEmpty) throw Exception('No results for address');

        final result = results[0];
        final components = result['address_components'] as List<dynamic>;

        String city = '', state = '', country = '', zipCode = '';

        for (var c in components) {
          final types = List<String>.from(c['types']);
          if (types.contains('locality')) city = c['long_name'];
          if (types.contains('administrative_area_level_1')) state = c['long_name'];
          if (types.contains('country')) country = c['long_name'];
          if (types.contains('postal_code')) zipCode = c['long_name'];
        }

        final location = result['geometry']['location'];

        return AddressDetails(
          latitude: (location['lat'] as num).toDouble(),
          longitude: (location['lng'] as num).toDouble(),
          city: city,
          state: state,
          country: country,
          zipCode: zipCode,
        );
      }

      throw Exception('Geocoding failed: ${response.statusCode}');
    } on DioException catch (e) {
      throw Exception('Dio Error: ${e.message}');
    }
  }

  // ─── LatLng → Address ────────────────────────────────────────────────────

  Future<AddressDetails> getAddressFromLatLng({
    required double latitude,
    required double longitude,
  }) async {
    final url =
        'https://maps.googleapis.com/maps/api/geocode/json?latlng=$latitude,$longitude&key=$placesApiKey';

    try {
      final response = await dio.get(url);

      if (response.statusCode == 200) {
        final results = response.data['results'] as List<dynamic>;
        if (results.isEmpty) throw Exception('No results for coordinates');

        final result = results[0];
        final components = result['address_components'] as List<dynamic>;

        String street = '', subLocality = '', city = '', state = '',
            country = '', zipCode = '';

        for (var c in components) {
          final types = List<String>.from(c['types']);
          if (types.contains('street_number')) street = c['long_name'];
          if (types.contains('route')) street = '$street ${c['long_name']}'.trim();
          if (types.contains('sublocality_level_1') || types.contains('sublocality'))
            subLocality = c['long_name'];
          if (types.contains('locality')) city = c['long_name'];
          if (types.contains('administrative_area_level_1')) state = c['long_name'];
          if (types.contains('country')) country = c['long_name'];
          if (types.contains('postal_code')) zipCode = c['long_name'];
        }

        return AddressDetails(
          latitude: latitude,
          longitude: longitude,
          city: city,
          state: state,
          country: country,
          zipCode: zipCode,
          street: street,
          subLocality: subLocality,
        );
      }

      throw Exception('Reverse geocoding failed: ${response.statusCode}');
    } on DioException catch (e) {
      throw Exception('Dio Error: ${e.message}');
    }
  }

  // ─── Place Autocomplete ───────────────────────────────────────────────────

  Future<List<PlaceSuggestion>> getPlaceSuggestions({
    required String input,
    String countryCode = 'us',
  }) async {
    final url =
        'https://maps.googleapis.com/maps/api/place/autocomplete/json?input=${Uri.encodeComponent(input)}&components=country:$countryCode&key=$placesApiKey';

    try {
      final response = await dio.get(url);
      if (response.statusCode == 200) {
        final predictions = response.data['predictions'] as List<dynamic>;
        return predictions.map((p) => PlaceSuggestion.fromMap(p)).toList();
      }
    } on DioException catch (e) {
      debugPrint('Places error: ${e.message}');
    }

    return [];
  }
}

// ─── Models ───────────────────────────────────────────────────────────────────

class AddressDetails {
  final double latitude;
  final double longitude;
  final String city;
  final String state;
  final String country;
  final String zipCode;
  final String street;
  final String subLocality;

  const AddressDetails({
    required this.latitude,
    required this.longitude,
    required this.city,
    required this.state,
    required this.country,
    required this.zipCode,
    this.street = '',
    this.subLocality = '',
  });
}

class PlaceSuggestion {
  final String placeId;
  final String description;

  const PlaceSuggestion({required this.placeId, required this.description});

  factory PlaceSuggestion.fromMap(Map<String, dynamic> map) {
    return PlaceSuggestion(
      placeId: map['place_id'] ?? '',
      description: map['description'] ?? '',
    );
  }
}
```

---

## DI Registration

```dart
// lib/core/di/service_locator.dart

sl.registerLazySingleton<LocationService>(() => LocationService());

sl.registerFactory<LocationCubit>(() => LocationCubit(sl<LocationService>()));
```

---

## Usage — App Router (Auto-fetch on App Start)

```dart
// lib/core/routing/app_router.dart
GoRoute(
  path: HomeScreen.path,
  builder: (context, state) => BlocProvider(
    create: (_) => sl<LocationCubit>()..fetchLocation(),
    child: const HomeScreen(),
  ),
),
```

---

## Usage — Listen to Position in Screen

```dart
// lib/features/home/presentation/screens/home_screen.dart

BlocListener<LocationCubit, Position?>(
  listenWhen: (prev, curr) => prev != curr,
  listener: (context, position) {
    if (position != null) {
      context.read<WeatherCubit>().loadWeather(position);
      context.read<OutfitCubit>().loadSuggestions(
        latitude: position.latitude,
        longitude: position.longitude,
      );
    }
  },
  child: const HomeBody(),
)
```

---

## Usage — Get Position Anywhere

```dart
// Read current position from cubit state
final position = context.read<LocationCubit>().state;

if (position != null) {
  print('Lat: ${position.latitude}, Lng: ${position.longitude}');
}

// Manually re-fetch
await context.read<LocationCubit>().fetchLocation();
```

---

## Usage — Reverse Geocode in DataSource

```dart
class ProfileRemoteDataSourceImpl {
  final GeocodingService _geocodingService;

  Future<void> updateLocation(double lat, double lng) async {
    final address = await _geocodingService.getAddressFromLatLng(
      latitude: lat,
      longitude: lng,
    );

    await _apiService.request(
      EndPoints.updateLocation,
      DioMethod.put,
      params: {
        'city': address.city,
        'state': address.state,
        'country': address.country,
        'latitude': lat,
        'longitude': lng,
      },
    );
  }
}
```

---

## Platform Setup

### Android (`android/app/src/main/AndroidManifest.xml`)

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

### iOS (`ios/Runner/Info.plist`)

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>We need your location to show personalized suggestions.</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>We need your location to show personalized suggestions.</string>
```

---

## Rules

- `LocationService` returns `null` on failure — never throw, always handle null
- Always check `isLocationServiceEnabled` before requesting permission
- `LocationCubit` state is `Position?` — null means location not available
- Never ask for location without a clear user-facing reason
- Use `LocationAccuracy.high` for precise location, `LocationAccuracy.low` for rough
- `GeocodingService` throws exceptions — catch in repository with `executeApiRequest`
- Always store Google Maps API key in `.env` / `AppConfig` — never hardcode
