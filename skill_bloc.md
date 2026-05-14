---
name: Flutter BLoC Skill
description: Production BLoC pattern extracted from the tailored app user feature. Covers Feature Bloc archetype only — events, state with _unset sentinel, typed handlers, session sync, merge helper, onTransition logging.
type: reference
---

# BLOC SKILL — tailored codebase (Bloc Only)

## Source
Extracted from `lib/features/user/presentation/bloc/` — UserBloc, UserEvent, UserState.

---

## THREE HANDLER PATTERNS

### 1. Sync fold (no await in success)
```dart
result.fold(
  (failure) => emit(state.copyWith(status: XStatus.error, message: failure.message)),
  (data) => emit(state.copyWith(status: XStatus.success, data: data)),
);
```

### 2. Async fold (session sync or any await in success)
```dart
await result.fold<Future<void>>(
  (failure) async => emit(state.copyWith(status: XStatus.error, message: failure.message)),
  (data) async {
    await _sessionCubit.updateUser(data);
    emit(state.copyWith(status: XStatus.success, data: data));
  },
);
```

### 3. Scoped sub-status (independent sub-operation)
```dart
emit(state.copyWith(blockedUsersStatus: BlockedUsersStatus.loading));
final result = await _useCase();
await result.fold<Future<void>>(
  (failure) async => emit(state.copyWith(blockedUsersStatus: BlockedUsersStatus.error, message: failure.message)),
  (data) async => emit(state.copyWith(blockedUsersStatus: BlockedUsersStatus.success, blockedUsers: data)),
);
```

---

## EVENT FILE SKELETON

```dart
import '../../domain/usecases/x_params.dart';

abstract class XEvent {
  const XEvent();
}

// No params
class GetXEvent extends XEvent {}

// With params
class CreateXEvent extends XEvent {
  final XParams params;
  const CreateXEvent(this.params);
}

// ID only
class DeleteXEvent extends XEvent {
  final String xId;
  const DeleteXEvent(this.xId);
}

// Always present
class ResetXStateEvent extends XEvent {
  const ResetXStateEvent();
}
```

---

## STATE FILE SKELETON

```dart
import 'package:equatable/equatable.dart';
import '../../domain/entities/x_entity.dart';

const _unset = Object();

// Primary status — initial, loading, success, error
enum XStatus { initial, loading, success, error }

// Secondary/scoped status — idle, loading, success, error
enum SubOperationStatus { idle, loading, success, error }

class XState extends Equatable {
  final XStatus status;
  final SubOperationStatus subStatus;
  final XEntity? data;
  final List<XEntity> items;
  final String? message;

  const XState({
    this.status = XStatus.initial,
    this.subStatus = SubOperationStatus.idle,
    this.data,
    this.items = const [],
    this.message,
  });

  XState copyWith({
    XStatus? status,
    SubOperationStatus? subStatus,
    Object? data = _unset,
    Object? items = _unset,
    Object? message = _unset,
  }) {
    return XState(
      status: status ?? this.status,
      subStatus: subStatus ?? this.subStatus,
      data: identical(data, _unset) ? this.data : data as XEntity?,
      items: identical(items, _unset)
          ? this.items
          : List<XEntity>.from(items as List<dynamic>),
      message: identical(message, _unset) ? this.message : message as String?,
    );
  }

  @override
  List<Object?> get props => [status, subStatus, data, items, message];
}
```

---

## BLOC FILE SKELETON

```dart
import 'dart:developer';

import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:tailored/core/session/session_cubit.dart';
import '../../domain/usecases/get_x_usecase.dart';
import '../../domain/usecases/create_x_usecase.dart';
import '../../domain/usecases/update_x_usecase.dart';
import '../../domain/entities/x_entity.dart';
import '../../domain/usecases/x_params.dart';
import 'x_event.dart';
import 'x_state.dart';

export 'x_event.dart';
export 'x_state.dart';

class XBloc extends Bloc<XEvent, XState> {
  final GetXUseCase _getXUseCase;
  final CreateXUseCase _createXUseCase;
  final UpdateXUseCase _updateXUseCase;
  final SessionCubit _sessionCubit;

  XBloc(
    this._getXUseCase,
    this._createXUseCase,
    this._updateXUseCase,
    this._sessionCubit,
  ) : super(const XState()) {
    on<GetXEvent>(_onGetX);
    on<CreateXEvent>(_onCreateX);
    on<UpdateXEvent>(_onUpdateX);
    on<ResetXStateEvent>(_onResetXState);
  }

  Future<void> _onGetX(GetXEvent event, Emitter<XState> emit) async {
    emit(state.copyWith(status: XStatus.loading, message: null));
    final result = await _getXUseCase();
    await result.fold<Future<void>>(
      (failure) async => emit(
        state.copyWith(status: XStatus.error, message: failure.message),
      ),
      (data) async {
        await _sessionCubit.updateUser(data);
        emit(state.copyWith(status: XStatus.success, data: data));
      },
    );
  }

  Future<void> _onCreateX(CreateXEvent event, Emitter<XState> emit) async {
    emit(state.copyWith(status: XStatus.loading, message: null));
    final result = await _createXUseCase(event.params);
    await result.fold<Future<void>>(
      (failure) async => emit(
        state.copyWith(status: XStatus.error, message: failure.message),
      ),
      (data) async {
        final merged = _mergeEntityWithParams(data, event.params);
        await _sessionCubit.updateUser(merged);
        emit(state.copyWith(status: XStatus.success, data: merged));
      },
    );
  }

  Future<void> _onUpdateX(UpdateXEvent event, Emitter<XState> emit) async {
    emit(state.copyWith(status: XStatus.loading, message: null));
    final result = await _updateXUseCase(event.params);
    await result.fold<Future<void>>(
      (failure) async => emit(
        state.copyWith(status: XStatus.error, message: failure.message),
      ),
      (data) async {
        final merged = _mergeEntityWithParams(data, event.params);
        await _sessionCubit.updateUser(merged);
        emit(state.copyWith(status: XStatus.success, data: merged));
      },
    );
  }

  // Prefer entity value; fallback to params when entity field is empty
  XEntity _mergeEntityWithParams(XEntity entity, XParams params) {
    return entity.copyWith(
      fieldA: entity.fieldA.trim().isNotEmpty ? entity.fieldA : params.fieldA,
    );
  }

  void _onResetXState(ResetXStateEvent event, Emitter<XState> emit) {
    emit(const XState());
  }

  @override
  void onTransition(Transition<XEvent, XState> transition) {
    log("event: ${transition.event},\nstate: ${transition.currentState}");
    super.onTransition(transition);
  }
}
```

---

## ARCHITECTURE RULES

1. One Bloc class per feature
2. All use cases injected as `final XUseCase _xUseCase` (private, prefixed with `_`)
3. `SessionCubit` injected when bloc must sync to global session after success
4. All handlers registered in constructor: `on<XEvent>(_onX)`
5. All handlers are private methods
6. Bloc file exports event + state: `export 'x_event.dart'; export 'x_state.dart';`
7. `onTransition` always overridden with `dart:developer` `log()` — never `print()`

---

## STATUS ENUM RULES

| Scope | Values |
|-------|--------|
| Primary operation | `initial, loading, success, error` |
| Secondary/scoped | `idle, loading, success, error` |

Each distinct sub-operation gets its own enum. Never reuse primary status for unrelated operations.

---

## STATE RULES

- Always `extends Equatable`
- `_unset` sentinel is **mandatory** for every nullable field in `copyWith`
- `message: null` passed at **every** loading emit — no exceptions
- List fields default to `const []`
- List fields in `copyWith` cast explicitly: `List<X>.from(x as List<dynamic>)`
- `props` lists every field

---

## FOLD RULE

| Success path | Use |
|-------------|-----|
| Synchronous | `result.fold(...)` |
| Has any `await` | `await result.fold<Future<void>>(...)` with `async` lambdas |

---

## MERGE RULE

When `create` or `update` API returns sparse data (doesn't echo back all fields):
- Add private `_mergeEntityWithParams(entity, params)` helper
- Prefer entity value when non-empty, fallback to params value
- Call merge before session sync and before emit

---

## VALIDATION CHECKLIST

- [ ] Every nullable field in `copyWith` uses `_unset` sentinel
- [ ] `message: null` at every loading emit
- [ ] Correct fold variant chosen (sync vs async)
- [ ] Primary enum: `initial/loading/success/error`
- [ ] Secondary enum: `idle/loading/success/error`
- [ ] `onTransition` overridden with `dart:developer` `log()`
- [ ] Bloc exports event + state files
- [ ] `ResetXStateEvent` + handler present
- [ ] List copyWith uses `List<X>.from(x as List<dynamic>)`
- [ ] Merge helper used when create/update API returns sparse data

---

## SKILL JSON

```json
{
  "skill": "bloc",
  "version": "1.1.0",
  "source_feature": "user",
  "scope": "bloc_only",
  "archetype": "FeatureBloc",
  "base": "Bloc<XEvent, XState>",
  "state_rules": {
    "extends": "Equatable",
    "unset_sentinel": true,
    "clear_message_on_loading": true,
    "list_defaults_to_const_empty": true,
    "primary_enum_values": ["initial", "loading", "success", "error"],
    "secondary_enum_values": ["idle", "loading", "success", "error"],
    "typed_list_cast_in_copyWith": true
  },
  "event_rules": {
    "abstract_base_const_constructor": true,
    "always_has_reset_event": true
  },
  "handler_rules": {
    "all_private": true,
    "emit_loading_with_null_message_first": true,
    "fold_sync": "result.fold(...)",
    "fold_async": "await result.fold<Future<void>>(...)",
    "merge_helper_on_create_update": true
  },
  "logging": {
    "package": "dart:developer",
    "method": "log",
    "override": "onTransition",
    "format": "event: ${transition.event},\\nstate: ${transition.currentState}"
  },
  "exports": {
    "bloc_exports_event_and_state": true
  }
}
```
