# Skeletonizer Loading Rules

This project uses `skeletonizer` package for all loading states in UI.
Never use `CircularProgressIndicator` or any spinner for screen/list/card loading.

---

## Package

```yaml
dependencies:
  skeletonizer: ^2.1.3
```

---

## Core Rule

- **NEVER** use `CircularProgressIndicator` for loading UI content
- **NEVER** use `LinearProgressIndicator` for loading UI content
- **NEVER** show an empty screen while data loads
- **ALWAYS** use `Skeletonizer` wrapping the real UI with `enabled: isLoading`
- The real widget renders inside Skeletonizer — skeleton effect shows on top when `enabled: true`
- Use `Loader.show()` / `Loader.during()` only for action buttons (POST/PUT/DELETE) — NOT for screen data loading

---

## Basic Pattern

```dart
Skeletonizer(
  enabled: isLoading,   // true = show skeleton, false = show real content
  child: YourRealWidget(),
)
```

The child is always the **real UI** — never a placeholder. Skeletonizer draws shimmer over it automatically.

---

## Pattern 1 — Single Card / Widget

```dart
BlocBuilder<FeatureCubit, FeatureState>(
  builder: (context, state) {
    final isLoading = state.status == FeatureStatus.loading;

    return Skeletonizer(
      enabled: isLoading,
      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(18),
        ),
        child: Column(
          children: [
            Text(isLoading ? 'Loading title...' : state.data?.title ?? ''),
            Text(isLoading ? 'Loading subtitle' : state.data?.subtitle ?? ''),
          ],
        ),
      ),
    );
  },
)
```

---

## Pattern 2 — List with Skeleton Items

```dart
BlocBuilder<FeatureBloc, FeatureState>(
  builder: (context, state) {
    final isLoading = state.status == FeatureStatus.loading;
    final items = isLoading
        ? List.generate(8, (_) => FeatureEntity.empty()) // fake items for skeleton
        : state.items;

    return Skeletonizer(
      enabled: isLoading,
      child: ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) => FeatureItemTile(item: items[index]),
      ),
    );
  },
)
```

> Always generate fake/empty items for list skeleton — Skeletonizer needs real widgets to draw shimmer over.

---

## Pattern 3 — List with Separate Loading Widget (Recommended for Lists)

```dart
BlocBuilder<FeatureBloc, FeatureState>(
  builder: (context, state) {
    final isLoading = state.status == FeatureStatus.loading;

    return Skeletonizer(
      enabled: isLoading,
      child: isLoading
          ? ListView.builder(
              itemCount: 8,
              itemBuilder: (_, __) => const _LoadingTile(), // skeleton placeholder tile
            )
          : ListView.builder(
              itemCount: state.items.length,
              itemBuilder: (_, index) => FeatureTile(item: state.items[index]),
            ),
    );
  },
)

// Skeleton placeholder tile — same structure as real tile
class _LoadingTile extends StatelessWidget {
  const _LoadingTile();

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: const CircleAvatar(),          // will be shimmed
      title: const Text('Loading name'),       // will be shimmed
      subtitle: const Text('Loading detail'),  // will be shimmed
    );
  }
}
```

---

## Pattern 4 — containersColor for Dark Backgrounds

```dart
Skeletonizer(
  enabled: isLoading,
  containersColor: context.theme.cardColor.withValues(alpha: 0.72),
  child: YourWidget(),
)
```

Use `containersColor` when the background is dark or colored — prevents white flash.

---

## Pattern 5 — RefreshIndicator + Skeletonizer

```dart
RefreshIndicator(
  onRefresh: () => context.read<FeatureCubit>().reload(),
  child: Skeletonizer(
    enabled: isLoading,
    child: ListView(
      physics: const AlwaysScrollableScrollPhysics(),
      children: [...],
    ),
  ),
)
```

Always put `Skeletonizer` **inside** `RefreshIndicator`.

---

## Skeleton Placeholder Values

When generating fake data for skeleton, use dummy strings that match the expected content length:

```dart
// Good — matches real content structure
factory FeatureEntity.empty() => const FeatureEntity(
  id: '',
  title: 'Loading title here',
  subtitle: 'Loading subtitle',
  imageUrl: '',
  count: 0,
);
```

---

## DO NOT

```dart
// ❌ Wrong — never use spinner for screen loading
if (state.isLoading) return const Center(child: CircularProgressIndicator());

// ❌ Wrong — never use spinner inside cards
if (isLoading) return const CircularProgressIndicator();

// ❌ Wrong — never show empty widget while loading
if (state.isLoading) return const SizedBox();

// ❌ Wrong — never use Loader.show() for screen data loading
Loader.show();
await cubit.loadData();
Loader.hide();
```

---

## Correct Full Screen Example

```dart
class FeatureScreen extends StatelessWidget {
  const FeatureScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Feature')),
      body: BlocBuilder<FeatureCubit, FeatureState>(
        builder: (context, state) {
          final isLoading = state.status == FeatureStatus.loading;
          final items = isLoading
              ? List.generate(6, (_) => FeatureEntity.empty())
              : state.items;

          return Skeletonizer(
            enabled: isLoading,
            child: ListView.builder(
              padding: const EdgeInsets.all(16),
              itemCount: items.length,
              itemBuilder: (context, index) => FeatureTile(item: items[index]),
            ),
          );
        },
      ),
    );
  }
}
```

---

## Summary Table

| Scenario | Use |
|---|---|
| Screen data loading | `Skeletonizer(enabled: isLoading)` |
| Card / widget loading | `Skeletonizer(enabled: isLoading)` |
| List loading | `Skeletonizer` + fake empty items |
| Button action (POST/PUT/DELETE) | `Loader.during()` or `Loader.show/hide()` |
| Form submit | `Loader.during()` |
| Any spinner / circle | NEVER |
