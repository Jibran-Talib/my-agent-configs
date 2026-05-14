# Coding Rules

## Clean architecture first
- Keep feature logic in `domain`, remote source in `data`, and UI in `presentation`.
- Avoid placing business logic in widgets.

## Naming and structure
- `UseCase` method name: `call()`.
- Repository implementation: `FeatureRepositoryImpl` implements `FeatureRepository`.
- Data source implementation: `FeatureRemoteDataSourceImpl`.
- Model<->Entity mappers are required.

## API call behavior
- Use `ApiService` with `DioMethod` enums.
- Use `executeApiRequest` helper in repositories for `Either<Failure, T>`.
- Data source throws exceptions (e.g., `BadRequestException`) if response not success.
- Repositories map to `Either` and forward for UI via Cubits.

## Error handling
- Use core failures in `core/errors/failure.dart`.
- Use readable message from API response when available.
- Leverage `ResponseWidget` for quick feedback in UI.

## UI consistency
- Use `AppTheme` from `core/theme/app_theme.dart`.
- Use `context.textTheme` and `context.colors` accessors from `theme_extensions.dart`.
- Reuse shared widgets (`ButtonComponent`, `TextComponent`, `Loader`, etc.).

## State changes
- Set status to loading before async call.
- On success set success status and payload.
- On error set error status and message.

## DI rules
- Register dependencies as lazy singletons in `core/di/service_locator.dart`.
- Register feature DI in each feature `*_injection.dart`.
- Cubit/Bloc should be registered as `registerFactory`.
