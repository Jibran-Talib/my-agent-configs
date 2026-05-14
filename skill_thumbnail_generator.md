---
name: Flutter Thumbnail Generator Skill
description: Reusable thumbnail generator service for Flutter projects — video thumbnail generation with in-memory cache, disk cache, FutureBuilder widget pattern, DI registration. Extracted from cier_check_user production app.
type: project
---

# FLUTTER THUMBNAIL GENERATOR SKILL (Global)

## WHEN TO APPLY
Any Flutter feature that displays video thumbnails — feed cards, video lists, media grids, video players.

---

## HOW IT WORKS

Three-layer cache + fallback:
1. **Memory cache** — `Map<String, String>` keyed by video URL/path → thumbnail file path
2. **Disk cache** — `getTemporaryDirectory()` + `hashCode` filename — survives hot restart, cleared by OS
3. **Generate** — `VideoThumbnail.thumbnailFile()` writes JPEG to temp dir
4. **Fallback** — any error returns `null`; widget shows broken-image icon

---

## SERVICE

```dart
// lib/core/services/thumbnail_generator_service.dart
import 'dart:io';
import 'package:flutter/foundation.dart';
import 'package:path_provider/path_provider.dart';
import 'package:video_thumbnail/video_thumbnail.dart';

class ThumbnailGeneratorService {
  // In-memory cache: videoUrl → local file path
  final Map<String, String> _memoryCache = {};

  Future<String?> getThumbnail(String videoUrlOrPath) async {
    // 1. Memory hit
    if (_memoryCache.containsKey(videoUrlOrPath)) {
      return _memoryCache[videoUrlOrPath];
    }

    try {
      final cacheDir = await getTemporaryDirectory();
      final fileName = videoUrlOrPath.hashCode.toString();
      final filePath = '${cacheDir.path}/$fileName.jpg';

      // 2. Disk hit
      if (File(filePath).existsSync()) {
        _memoryCache[videoUrlOrPath] = filePath;
        return filePath;
      }

      // 3. Generate
      final thumb = await VideoThumbnail.thumbnailFile(
        video: videoUrlOrPath,
        thumbnailPath: filePath,
        imageFormat: ImageFormat.JPEG,
        maxHeight: 180,   // adapt to your card height
        quality: 75,      // 75 is a good balance size/quality
      );

      if (thumb != null) {
        _memoryCache[videoUrlOrPath] = thumb;
      }

      return thumb;
    } catch (e) {
      debugPrint('Thumbnail error: $e');
      return null;
    }
  }
}
```

> **Adapt:** Change `maxHeight` and `quality` per design. Keep `ImageFormat.JPEG` for smallest file size.

---

## DI REGISTRATION

Register as `LazySingleton` so the memory cache is shared across all widgets:

```dart
// In lib/core/services/service_locator.dart
locator.registerLazySingleton<ThumbnailGeneratorService>(
  () => ThumbnailGeneratorService(),
);
```

> **Never** register as `Factory` — you'd lose the memory cache on every widget build.

---

## WIDGET — THUMBNAIL WITH PLAY ICON (generic, reusable)

Use `FutureBuilder<String?>` wrapping `getThumbnail()`. Always show loading skeleton and fallback.

```dart
// lib/shared/widgets/video_thumbnail_widget.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:skeletonizer/skeletonizer.dart';
import 'package:your_app/core/services/service_locator.dart';
import 'package:your_app/core/services/thumbnail_generator_service.dart';

class VideoThumbnailWidget extends StatelessWidget {
  final String videoUrl;
  final double? width;
  final double? height;
  final BoxFit fit;

  VideoThumbnailWidget({
    super.key,
    required this.videoUrl,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
  });

  // Access singleton via locator — never constructor-injected
  final _thumbnailService = locator<ThumbnailGeneratorService>();

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String?>(
      future: _thumbnailService.getThumbnail(videoUrl),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return _buildLoading();
        }

        if (snapshot.hasError || snapshot.data == null) {
          return _buildFallback();
        }

        return Stack(
          fit: StackFit.expand,
          children: [
            Image.file(
              File(snapshot.data!),
              width: width,
              height: height,
              fit: fit,
            ),
            // Play icon overlay
            Container(
              color: Colors.black26,
              child: const Center(
                child: Icon(
                  Icons.play_circle_fill,
                  size: 40,
                  color: Colors.white,
                ),
              ),
            ),
          ],
        );
      },
    );
  }

  Widget _buildLoading() {
    return Skeletonizer(
      enabled: true,
      child: Container(
        width: width ?? double.infinity,
        height: height ?? double.infinity,
        decoration: BoxDecoration(borderRadius: BorderRadius.circular(8)),
      ),
    );
  }

  Widget _buildFallback() {
    return Container(
      width: width,
      height: height,
      color: Colors.grey[300],
      child: const Center(
        child: Icon(Icons.broken_image, size: 30, color: Colors.grey),
      ),
    );
  }
}
```

---

## WIDGET — MEDIA ITEM (image OR video, from entity)

When you have a `MediaEntity` with a `type` field:

```dart
// lib/shared/widgets/feed_card/feed_media_item.dart
import 'dart:io';
import 'package:cached_network_image/cached_network_image.dart';
import 'package:flutter/material.dart';
import 'package:skeletonizer/skeletonizer.dart';
import 'package:your_app/core/services/service_locator.dart';
import 'package:your_app/core/services/thumbnail_generator_service.dart';
import 'package:your_app/features/feeds/domain/entities/feed_entity.dart';

class FeedMediaItemThumbnail extends StatelessWidget {
  final MediaEntity media;

  FeedMediaItemThumbnail({super.key, required this.media});

  final thumbnailService = locator<ThumbnailGeneratorService>();

  @override
  Widget build(BuildContext context) {
    if (media.type == MediaType.image) {
      return _buildImage(media.url);
    }
    return _buildVideoThumbnail(media.url);
  }

  Widget _buildImage(String url) {
    return CachedNetworkImage(
      imageUrl: url,
      fit: BoxFit.cover,
      placeholder: (_, __) => _buildLoading(),
      errorWidget: (_, __, ___) => _buildFallback(),
    );
  }

  Widget _buildVideoThumbnail(String url) {
    return FutureBuilder<String?>(
      future: thumbnailService.getThumbnail(url),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return _buildLoading();
        }
        if (snapshot.hasError || snapshot.data == null) {
          return _buildFallback();
        }
        return Stack(
          fit: StackFit.expand,
          children: [
            Image.file(File(snapshot.data!), fit: BoxFit.cover),
            Container(
              color: Colors.black26,
              child: const Center(
                child: Icon(Icons.play_circle_fill, size: 40, color: Colors.white),
              ),
            ),
          ],
        );
      },
    );
  }

  Widget _buildFallback() {
    return Container(
      color: Colors.grey[300],
      child: const Center(child: Icon(Icons.broken_image, size: 30)),
    );
  }

  Widget _buildLoading() {
    return Skeletonizer(
      enabled: true,
      child: Container(
        width: double.infinity,
        height: double.infinity,
        decoration: BoxDecoration(borderRadius: BorderRadius.circular(8)),
      ),
    );
  }
}
```

---

## WIDGET — VIDEO CARD (title + description + thumbnail)

For list cards that show thumbnail + metadata:

```dart
// lib/shared/widgets/video_card_widget.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';
import 'package:skeletonizer/skeletonizer.dart';
import 'package:your_app/core/services/service_locator.dart';
import 'package:your_app/core/services/thumbnail_generator_service.dart';
import 'package:your_app/shared/widgets/text_component.dart';
import 'package:your_app/core/constants/app_colors.dart';
import 'package:your_app/core/utils/spacing.dart';

class VideoCardWidget extends StatelessWidget {
  final String mediaUrl;
  final String title;
  final String description;
  final bool isLoading;

  const VideoCardWidget({
    super.key,
    required this.mediaUrl,
    required this.title,
    required this.description,
    this.isLoading = false,
  });

  @override
  Widget build(BuildContext context) {
    final thumbnailService = locator<ThumbnailGeneratorService>();
    return Skeletonizer(
      enabled: isLoading,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          ClipRRect(
            borderRadius: BorderRadius.circular(20.r),
            child: SizedBox(
              width: double.infinity,
              height: 187.h,
              child: Stack(
                children: [
                  Positioned.fill(
                    child: Skeleton.replace(
                      width: double.infinity,
                      height: 187.h,
                      child: isLoading
                          ? Container(color: AppColors.grey.withOpacity(0.2))
                          : FutureBuilder<String?>(
                              future: thumbnailService.getThumbnail(mediaUrl),
                              builder: (context, snapshot) {
                                if (snapshot.connectionState ==
                                    ConnectionState.waiting) {
                                  return const Center(
                                    child: CircularProgressIndicator(),
                                  );
                                }
                                if (snapshot.data == null) {
                                  return Container(color: Colors.grey[300]);
                                }
                                return Stack(
                                  fit: StackFit.expand,
                                  children: [
                                    Image.file(
                                      File(snapshot.data!),
                                      fit: BoxFit.cover,
                                    ),
                                    Container(color: Colors.black26),
                                  ],
                                );
                              },
                            ),
                    ),
                  ),
                  Center(
                    child: CircleAvatar(
                      radius: 20.r,
                      backgroundColor: Colors.black38,
                      child: const Icon(
                        Icons.play_arrow,
                        color: Colors.white,
                        size: 24,
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
          VerticalSpacing(15),
          TextComponent(
            text: title,
            fontsize: 14.sp,
            fontWeight: FontWeight.w600,
          ),
          VerticalSpacing(5),
          TextComponent(
            text: description,
            fontsize: 12.sp,
            fontWeight: FontWeight.w400,
            color: AppColors.textPrimary.withOpacity(0.8),
          ),
        ],
      ),
    );
  }
}
```

---

## GENERATION STEPS (in order)

1. **Add dependency** — `video_thumbnail` + `path_provider` to `pubspec.yaml`
2. **Service** — create `ThumbnailGeneratorService` in `lib/core/services/thumbnail_generator_service.dart`
3. **DI** — register `LazySingleton<ThumbnailGeneratorService>` in `service_locator.dart`
4. **Generic widget** (optional) — create `VideoThumbnailWidget` in `shared/widgets/`
5. **Feature widget** — use `FutureBuilder<String?>` wrapping `locator<ThumbnailGeneratorService>().getThumbnail(url)` in any widget that shows a video
6. **Always** handle three states: `waiting → skeleton`, `null/error → fallback icon`, `done → Image.file + play overlay`

---

## KEY RULES

| Rule | Detail |
|---|---|
| DI | `registerLazySingleton` — NEVER `registerFactory` (would lose memory cache) |
| Access | `locator<ThumbnailGeneratorService>()` in widget — never constructor-injected |
| Null safety | `snapshot.data == null` always goes to fallback — never force-unwrap |
| File display | `Image.file(File(snapshot.data!))` — path returned is always a local file path |
| Cache key | `videoUrlOrPath.hashCode.toString()` — unique per URL, stable across restarts |
| Format | Always `ImageFormat.JPEG` — smallest size, fastest decode |
| maxHeight | Match to your card height (default 180) — larger = slower + bigger file |
| quality | 75 is production default — lower for lists, higher for detail screens |
| Skeletonizer | Wrap loading state with `Skeletonizer(enabled: true)` — never `CircularProgressIndicator` alone |
| Error logging | `debugPrint('Thumbnail error: $e')` only — never rethrow, always return null |

---

## PUBSPEC DEPENDENCIES

```yaml
dependencies:
  video_thumbnail: ^0.5.3      # thumbnail generation
  path_provider: ^2.1.2        # getTemporaryDirectory()
  cached_network_image: ^3.4.1 # for image-type media
  skeletonizer: ^1.4.2         # loading skeleton
  flutter_screenutil: ^5.9.3   # .w / .h / .r sizing
```

---

## ANDROID SETUP

`video_thumbnail` requires no special Android permissions for remote URLs.
For local video files, ensure `READ_EXTERNAL_STORAGE` (Android < 13) or `READ_MEDIA_VIDEO` (Android 13+) is declared in `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO"/>
```

---

## VALIDATION CHECKLIST

- [ ] `ThumbnailGeneratorService` registered as `LazySingleton` (not Factory)
- [ ] Memory cache `Map<String, String>` is instance-level (not static)
- [ ] Disk cache check `File(filePath).existsSync()` before generating
- [ ] `VideoThumbnail.thumbnailFile()` used (not `thumbnailData`) — returns file path
- [ ] `ImageFormat.JPEG` set explicitly
- [ ] `try/catch` wraps entire generation block
- [ ] `debugPrint` on error, returns `null` — never throws
- [ ] Widget uses `FutureBuilder<String?>` (nullable)
- [ ] Three builder states handled: `waiting`, `null/error`, `done`
- [ ] `Image.file(File(snapshot.data!))` — correct local file rendering
- [ ] Play icon overlay shown on video thumbnails
- [ ] Loading state uses `Skeletonizer` not raw `CircularProgressIndicator`
- [ ] Fallback shows grey container + `Icons.broken_image`
- [ ] `locator<ThumbnailGeneratorService>()` used in widget — NOT constructor param

---

**Source:** Extracted from production `cier_check_user` Flutter app — `core/services/thumbnail_generator_service.dart` + `shared/widgets/feed_card/feed_media_item.dart` + `features/doctors/presentation/widgets/video_card_widget.dart`.  
**How to apply:** Follow Generation Steps 1→6. Validate against checklist before marking complete.
