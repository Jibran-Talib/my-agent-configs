# Project Architecture

## Overview
- Architecture style: Clean architecture with feature-based module separation.
- Dependency injection: `GetIt` (located in `lib/core/di/service_locator.dart`).
- Routing: `go_router` in `lib/core/routing/app_router.dart` + feature route files (`features/auth/auth_routes.dart`, `features/user/user_routes.dart`).
- State management: `flutter_bloc` (Cubit + Bloc patterns).
- Network layer: Dio wrapped by `ApiService`, with interceptors and centralized exception mapping.

## Root folder layout
- `lib/core`: shared infrastructure, configuration, theme, services, API layer, errors, utilities.
- `lib/features`: feature modules (`auth`, `avatar`, `config`, `user`, `main`), each with `data`, `domain`, `presentation`, and optionally `di`/`routes`.
- `lib/shared`: reusable widgets and models (e.g., `text_component.dart`, `button_component.dart`, `response_widget.dart`).

## Feature module pattern
Each feature (e.g., `auth`, `user`) follows:
1. `data/datasources`: remote/data access (e.g., `auth_remote_datasource.dart`).
2. `data/repositories`: repository implementations using `executeApiRequest` and data source.
3. `domain/entities`: immutable business entities.
4. `domain/models`: typically in `data/models`, with `toEntity()` and `fromEntity()` mapping.
5. `domain/repositories`: abstract repository interfaces.
6. `domain/usecases`: use case classes with `call()`.
7. `presentation`: `cubit`/`bloc` + screens + widgets.

## DI flow
- `initDependencies()` is called to register core + feature DI.
- Feature injection registers datasources, repositories, usecases, cubits/blocs.
- Cubits and blocs are injected into UI via `BlocProvider` or manually in routes.

## API flow
1. UI triggers Cubit/Bloc action.
2. Cubit/Bloc calls UseCase.
3. UseCase calls Repository.
4. Repository executes `executeApiRequest` with data source function.
5. DataSource calls `ApiService.request` with `EndPoints` and `DioMethod`.
6. `ApiService` returns `ApiResponse`; repository converts to `Entity`.
7. Cubit/Bloc emits state according to `Either<Failure, T>` result.

## Error handling
- Exceptions are converted to `Failure` in `core/api_service/api_handler.dart`.
- `core/errors` contains typed failures (`NetworkFailure`, `UnauthorizedFailure`, etc.).
- UI components usually observe state status and message.

## State persists
- `SessionCubit` handles session token+user storage and restoration using `StorageService`.
- Auth is saved in local storage on login.
