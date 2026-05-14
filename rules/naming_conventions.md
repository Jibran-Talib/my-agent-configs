# Naming Conventions

## File naming
- Use `snake_case` for files.
- Align directories with features and layers:
  - `features/auth/data/models/auth_model.dart`
  - `features/auth/domain/usecases/login_usecase.dart`
  - `features/auth/presentation/cubit/auth_cubit.dart`

## Class names
- PascalCase for all classes and enums.
- Use `Feature` prefix when appropriate.
  - `AuthCubit`, `UserBloc`, `AppTheme`, `ApiService`, `AppConfig`.

## Variables and methods
- camelCase for variables and methods.
- Use descriptive names: `getUser`, `createUser`, `updateUser`, `checkUsername`.

## Enums
- PascalCase for enum names and values.
  - `AuthStatus.initial`, `UserStatus.loading`, `DioMethod.post`.

## Constants
- Uppercase with underscore for global constants.
  - `kBaseUrl`, `kImageBaseUrl`.
- Class-based constants for endpoints:
  - `EndPoints.login`, `EndPoints.profile`.

## State properties
- `status`, `action`, `message`, `user`, `auth`, `isUsernameAvailable`, etc.
- `copyWith` arguments use nullable semantics with `_unset` sentinel when needed.

## Usecase classes
- `[Feature]UseCase` (e.g., `GetUserUseCase`, `LoginUseCase`).
- Optional params classes: `[Feature]Params` or `[Feature]RequestParams`.
