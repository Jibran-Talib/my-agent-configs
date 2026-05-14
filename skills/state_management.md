# State Management Pattern

## Primary library
- `flutter_bloc` (Cubit/Bloc) is the official pattern used throughout the app.
- Main providers are configured in `lib/main.dart` with `MultiBlocProvider`.

## Cubit conventions
- Files under `features/*/presentation/cubit`.
- Cubit class name: `[Feature]Cubit` (e.g., `AuthCubit`, `AvatarCubit`, `ConfigCubit`, `DraftCubit`).
- State class name: `[Feature]State` with `Equatable` or plain class.
- Status enum for process tracking (e.g., `AuthStatus`, `UserStatus`).
- `copyWith` is used to update state immutably.
- Each method sets `loading`, calls usecase then emits `success` or `error`.

## Bloc conventions
- Files under `features/*/presentation/bloc`.
- Bloc class name: `[Feature]Bloc` (e.g., `UserBloc`).
- Event class: `[Feature]Event` with subclasses (`GetUserEvent`, etc.).
- State class: `[Feature]State` with status and payload.
- `on<Event>` handlers are used and emit state accordingly.

## Session handling
- `SessionCubit` under `core/session` handles login session, user persistence, logout.
- It stores token/user in `StorageService`.

## UI integration
- Screens observe Cubit/Bloc state with `BlocBuilder`, `BlocListener`, `BlocConsumer`.
- Global app routes instantiate blocs/cubits with `BlocProvider` in `app_router.dart`.

## Shared local state
- Some feature-specific state is managed by Cubit (e.g., `DraftCubit`) without API.
- Combines local updates and API actions in one Cubit.
