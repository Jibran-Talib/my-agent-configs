---
name: Flutter Pagination Skill
description: Complete reusable pagination skill for Flutter Clean Architecture + BLoC projects — guards, DataSource, Repository, UseCase, UI scroll pattern
type: project
---

# FLUTTER PAGINATION SKILL (Global)

## WHEN TO APPLY
Any Flutter feature that shows a list with infinite scroll / load-more / paged API.

---

## CORE CLASSES (check if exist, create if missing)

### PaginationEntity (domain)
```dart
class PaginationEntity {
  final int total;
  final int page;
  final int limit;
  final int? nextPage;
  final bool hasMore;

  const PaginationEntity({
    required this.total,
    required this.page,
    required this.limit,
    required this.nextPage,
    required this.hasMore,
  });
}
```

### Pagination Model (data)
```dart
class Pagination {
  final int total;
  final int page;
  final int limit;
  final int? nextPage;
  final bool hasMore;

  const Pagination({...});

  factory Pagination.fromJson(Map<String, dynamic> json) {
    return Pagination(
      total: json['total'] ?? 0,
      page: json['page'] ?? 1,
      limit: json['limit'] ?? 25,
      nextPage: json['nextPage'],
      hasMore: json['hasMore'] ?? false,
    );
  }

  PaginationEntity toEntity() => PaginationEntity(
    total: total, page: page, limit: limit,
    nextPage: nextPage, hasMore: hasMore,
  );
}
```

### PaginationParams
```dart
class PaginationParams {
  final int page;
  final int limit;
  const PaginationParams({required this.page, required this.limit});
  Map<String, dynamic> toJson() => {'page': page, 'limit': limit};
}
```

---

## NAMING CONVENTIONS

| Thing | Pattern |
|---|---|
| Fetch Event | `FetchXxx({bool isInitial = false, required PaginationParams params})` |
| Clear Event | `ClearXxxEvent` |
| States | `XxxInitial / XxxLoading / XxxLoaded / XxxError` |
| Paginated Model | `PaginatedXxxModel` |
| Paginated Entity | `PaginatedXxxEntity` |
| DataSource abstract | `XxxDataSource` |
| DataSource impl | `XxxDataSourceImpl` |
| Repository abstract | `XxxRepository` |
| Repository impl | `XxxRepositoryImpl` |

---

## BLOC PATTERN

### Guard Fields (always 3)
```dart
bool _hasMore = true;
bool _isFetching = false;
int _page = 1;
int get currentPage => _page;
```

### Fetch Handler
```dart
Future<void> _fetchXxx(FetchXxx event, Emitter<XxxState> emit) async {
  if (_isFetching) return;
  if (!_hasMore && !event.isInitial) return;

  _isFetching = true;

  if (event.isInitial) {
    emit(XxxLoading());
    _items.clear();
    _page = 1;
    _hasMore = true;
    _isFetching = false; // reset before async
  }

  final result = await _usecase(event.params);

  result.fold(
    (failure) => emit(XxxError(failure.message)),
    (res) {
      _items.addAll(res.items);       // addAll — never reassign
      _page++;
      _hasMore = res.pagination.hasMore;
      emit(XxxLoaded(
        items: List.from(_items),     // always new copy
        hasMore: _hasMore,
      ));
    },
  );

  _isFetching = false;
}
```

### Clear Event
```dart
on<ClearXxxEvent>((event, emit) {
  emit(XxxLoading());
  _items.clear();
  _hasMore = true;
  _isFetching = false;
  _page = 1;
});
```

---

## STATES

```dart
abstract class XxxState {}

class XxxInitial extends XxxState {}
class XxxLoading extends XxxState {}
class XxxError extends XxxState {
  final String message;
  XxxError(this.message);
}

class XxxLoaded extends XxxState {
  final List<XxxEntity> items;
  final bool hasMore;
  final bool isLoadingMore;

  XxxLoaded({
    required this.items,
    this.hasMore = false,
    this.isLoadingMore = false,
  });

  XxxLoaded copyWith({
    List<XxxEntity>? items,
    bool? hasMore,
    bool? isLoadingMore,
  }) {
    return XxxLoaded(
      items: items ?? this.items,
      hasMore: hasMore ?? this.hasMore,
      isLoadingMore: isLoadingMore ?? this.isLoadingMore,
    );
  }
}
```

---

## DATA SOURCE PATTERN

```dart
abstract class XxxDataSource {
  Future<Either<Failure, PaginatedXxxModel>> getXxx(PaginationParams params);
}

class XxxDataSourceImpl implements XxxDataSource {
  // Use service locator — never constructor inject DioApiService
  final DioApiService apiService = locator<DioApiService>();

  @override
  Future<Either<Failure, PaginatedXxxModel>> getXxx(PaginationParams params) async {
    try {
      ApiResponse res = await apiService.get(
        ApiEndpoints.xxx,
        queryParameters: {"page": params.page, "limit": params.limit},
      );
      return Right(PaginatedXxxModel.fromJson(res.data));
    } on ApiException catch (e) {
      debugPrint("Error in getXxx: $e");
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      debugPrint("Unexpected error in getXxx: $e");
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }
}
```

---

## REPOSITORY PATTERN

```dart
class XxxRepositoryImpl implements XxxRepository {
  final XxxDataSource dataSource;
  const XxxRepositoryImpl(this.dataSource);

  @override
  Future<Either<Failure, PaginatedXxxEntity>> getXxx({
    required PaginationParams params,
  }) async {
    final response = await dataSource.getXxx(params);
    return response.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (res) => Right(res.toEntity()),
    );
  }
}
```

---

## PAGINATED MODEL PATTERN

```dart
class PaginatedXxxModel {
  final List<XxxModel> items;
  final Pagination pagination;

  const PaginatedXxxModel({required this.items, required this.pagination});

  factory PaginatedXxxModel.fromJson(Map<String, dynamic> json) {
    return PaginatedXxxModel(
      items: (json['data'] as List<dynamic>? ?? [])
          .map((e) => XxxModel.fromJson(e as Map<String, dynamic>))
          .toList(),
      pagination: Pagination.fromJson(json['pagination'] ?? {}),
    );
  }

  PaginatedXxxEntity toEntity() => PaginatedXxxEntity(
    items: items.map((e) => e.toEntity()).toList(),
    pagination: pagination.toEntity(),
  );
}
```

---

## UI PATTERN

```dart
// 1. Setup
late final ScrollController _scrollController;

@override
void initState() {
  super.initState();
  _scrollController = ScrollController()..addListener(_onScroll);
  context.read<XxxBloc>().add(FetchXxx(
    isInitial: true,
    params: PaginationParams(page: 1, limit: 25),
  ));
}

// 2. Scroll trigger
void _onScroll() {
  final max = _scrollController.position.maxScrollExtent;
  final current = _scrollController.position.pixels;
  if (max - current <= 200) {
    final bloc = context.read<XxxBloc>();
    if (bloc.state is XxxLoaded && (bloc.state as XxxLoaded).hasMore) {
      bloc.add(FetchXxx(
        params: PaginationParams(page: bloc.currentPage, limit: 25),
      ));
    }
  }
}

@override
void dispose() {
  _scrollController.dispose();
  super.dispose();
}

// 3. ListView
BlocBuilder<XxxBloc, XxxState>(
  builder: (context, state) {
    if (state is XxxLoading) return const XxxShimmer();
    if (state is XxxError) return Center(child: Text(state.message));
    if (state is XxxLoaded) {
      return RefreshIndicator(
        onRefresh: () async {
          context.read<XxxBloc>().add(FetchXxx(
            isInitial: true,
            params: PaginationParams(page: 1, limit: 25),
          ));
        },
        child: ListView.builder(
          controller: _scrollController,
          itemCount: state.items.length + (state.hasMore ? 1 : 0),
          itemBuilder: (context, index) {
            if (index == state.items.length) {
              return const Center(child: CircularProgressIndicator());
            }
            return XxxCard(item: state.items[index]);
          },
        ),
      );
    }
    return const SizedBox.shrink();
  },
)
```

---

## DEFAULTS
- Default limit: **25**
- Default page: **1**
- Scroll threshold: **200px** from bottom
- Error logging: **debugPrint** only
- Either: **dartz** package
- Failure: **ServerFailure(message: ...)**

---

## GENERATION STEPS (in order)
1. Core classes (PaginationEntity, Pagination, PaginationParams) — create if missing
2. `PaginatedXxxModel` — data layer
3. `PaginatedXxxEntity` — domain layer
4. `XxxDataSource` abstract + `XxxDataSourceImpl`
5. `XxxRepository` abstract + `XxxRepositoryImpl`
6. `XxxUseCase` accepting PaginationParams
7. Events: `FetchXxx(isInitial, params)` + `ClearXxxEvent`
8. States: Initial / Loading / Loaded(copyWith) / Error
9. Bloc with 3 guards + fetch handler + clear handler
10. UI: ScrollController + ListView + CircularProgressIndicator + RefreshIndicator

---

## VALIDATION CHECKLIST
- [ ] 3 guard fields in Bloc: _hasMore, _isFetching, _page
- [ ] isInitial flag on FetchXxx event
- [ ] ClearXxxEvent resets all guards
- [ ] _items.addAll() used (not reassign)
- [ ] List.from(_items) emitted (new copy)
- [ ] locator<DioApiService>() — not constructor injected
- [ ] queryParameters: {"page": ..., "limit": ...}
- [ ] ApiException caught before generic catch
- [ ] res.toEntity() in repository
- [ ] ServerFailure for all errors
- [ ] XxxLoaded has copyWith()
- [ ] itemCount = length + (hasMore ? 1 : 0)
- [ ] CircularProgressIndicator at last index
- [ ] RefreshIndicator dispatches isInitial: true
- [ ] ScrollController disposed

**Why:** This pattern was extracted from a production Flutter app (cier_check_user) where notifications and wallet features use this exact implementation.
**How to apply:** Follow every step in Generation Steps. Validate against checklist before finishing.
