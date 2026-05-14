---
name: Flutter Wallet Feature Skill
description: Complete reusable wallet/transaction feature skill for Flutter Clean Architecture + BLoC. Asks ONLY for API endpoints, request body, response shape. Generates everything else automatically — DI, routes, enums, params, widgets, dialogs, filters, business logic.
type: project
---

# FLUTTER WALLET FEATURE SKILL (Global)

## HOW TO USE THIS SKILL

When user asks to build a wallet/transaction/payment-history feature:

**ASK THE USER ONLY:**
1. API endpoint path(s) — e.g., `/wallet/transactions`
2. Request body/query params — e.g., `{ page, limit }`
3. API response shape — e.g., `{ result: [...], pagination: {...} }`
4. Field names in the response item — e.g., `_id, amount, kind, currency, status, createdAt`

**EVERYTHING ELSE IS GENERATED AUTOMATICALLY:**
- Folder structure
- All 3 clean architecture layers (data / domain / presentation)
- Enums + extensions for transaction types
- Entity with business logic helpers
- Model with fromJson / toJson / toEntity / fromEntity
- Paginated model + entity
- DataSource (abstract + impl)
- Repository (abstract + impl)
- UseCase
- BLoC events / states / bloc with 3 pagination guards
- Filter enum + helper class
- All UI private widgets (BalanceCard, TransactionsSection, FilterBottomSheet, FilterOption)
- Skeletonizer loading
- BlocConsumer with error snackbar
- ScrollController + infinite scroll
- RefreshIndicator
- Route enum entry
- GoRouter route registration
- GetIt DI registration

---

## FOLDER STRUCTURE

```
lib/features/wallet/
├── data/
│   ├── data_source/
│   │   └── wallet_transaction_history_data_source.dart
│   ├── models/
│   │   ├── transaction_history_model.dart
│   │   └── paginated_transaction_history_model.dart
│   └── repositories/
│       └── wallet_repository_impl.dart
├── domain/
│   ├── enitities/                          ← keep typo for project consistency
│   │   ├── transaction_history_entity.dart
│   │   └── paginated_transaction_history_entity.dart
│   ├── repositories/
│   │   └── wallet_repository.dart
│   └── usecase/
│       └── get_transaction_history_usecase.dart
└── presentation/
    ├── bloc/
    │   ├── transaction_history _bloc.dart   ← note: space in filename (keep for consistency)
    │   ├── transaction_history_events.dart
    │   └── transaction_history_states.dart
    └── screens/
        └── wallet_screen.dart
```

---

## STEP 1 — ENUM + EXTENSION PATTERN

Every transaction type gets an enum with a serialization extension.

```dart
// In transaction_history_model.dart
enum TransactionKind { received, refund, charge, other }

extension TransactionKindExtension on TransactionKind {
  String toJson() => name;

  static TransactionKind fromJson(String? kind) {
    return TransactionKind.values.firstWhere(
      (e) => e.name == kind,
      orElse: () => TransactionKind.other,
    );
  }
}
```

**Rule:** `orElse` always returns the safe default (`.other`, `.unknown`, `.none`, etc.)

---

## STEP 2 — DATA MODEL PATTERN

```dart
class TransactionHistoryModel {
  final String id;           // json key: '_id'
  final String user;
  final TransactionKind kind;
  final double? amount;      // stored in CENTS — divide by 100 for display
  final String? currency;
  final String? status;
  final DateTime createdAt;
  final DateTime updatedAt;
  // ... add more fields as per API response

  const TransactionHistoryModel({...});

  factory TransactionHistoryModel.fromJson(Map<String, dynamic> json) {
    return TransactionHistoryModel(
      id: json['_id'],
      kind: TransactionKindExtension.fromJson(json['kind']),
      amount: (json['amount'] as num?)?.toDouble(),
      currency: json['currency'],
      status: json['status'],
      createdAt: DateTime.parse(json['createdAt']),
      updatedAt: DateTime.parse(json['updatedAt']),
      // null-safe: json['field'] ?? defaultValue
    );
  }

  Map<String, dynamic> toJson() {
    return {
      '_id': id,
      'kind': kind.toJson(),
      if (amount != null) 'amount': amount,   // ← conditional include nulls
      if (currency != null) 'currency': currency,
      'createdAt': createdAt.toIso8601String(),
    };
  }

  TransactionHistoryEntity toEntity() => TransactionHistoryEntity(
    id: id, kind: kind, amount: amount, ...
  );

  factory TransactionHistoryModel.fromEntity(TransactionHistoryEntity entity) =>
    TransactionHistoryModel(id: entity.id, kind: entity.kind, ...);
}
```

**Rules:**
- Dates always `DateTime.parse(json['field'])`
- Numbers always `(json['amount'] as num?)?.toDouble()`
- Nullable fields use `if (field != null) 'key': field` in toJson
- Always include `fromEntity()` constructor

---

## STEP 3 — PAGINATED MODEL PATTERN

```dart
// data json key for list is usually 'result' or 'data' — confirm with API response
class PaginatedTransactionHistoryModel {
  final List<TransactionHistoryModel> transactionHistory;
  final Pagination pagination;

  PaginatedTransactionHistoryModel({
    required this.transactionHistory,
    required this.pagination,
  });

  factory PaginatedTransactionHistoryModel.fromJson(Map<String, dynamic> json) {
    return PaginatedTransactionHistoryModel(
      transactionHistory: (json['result'] as List<dynamic>)  // ← ASK user for key
          .map((x) => TransactionHistoryModel.fromJson(x))
          .toList(),
      pagination: Pagination.fromJson(json['pagination']),
    );
  }

  Map<String, dynamic> toJson() => {
    'result': transactionHistory.map((e) => e.toJson()).toList(),
    'pagination': pagination.toJson(),
  };

  PaginatedTransactionHistoryEntity toEntity() =>
    PaginatedTransactionHistoryEntity(
      transactionHistory: transactionHistory.map((e) => e.toEntity()).toList(),
      pagination: pagination.toEntity(),
    );
}
```

---

## STEP 4 — ENTITY WITH BUSINESS LOGIC PATTERN

Entity lives in domain layer — NO flutter imports — pure Dart.

```dart
import 'package:intl/intl.dart';
import '../data/models/transaction_history_model.dart'; // only for enum

class TransactionHistoryEntity {
  final String id;
  final TransactionKind kind;
  final double? amount;       // cents
  final String? currency;
  final String? status;
  final DateTime createdAt;
  final DateTime updatedAt;

  const TransactionHistoryEntity({...});

  // ─── BUSINESS HELPERS ─────────────────────────────────────────────────────

  String getFormattedAmount({bool withCurrency = false}) {
    if (amount == null) return 'N/A';
    final double dollars = amount! / 100.0;              // cents → dollars
    final formatter = NumberFormat('#,##0.00');
    if (withCurrency) return '${_currencySymbol()}${formatter.format(dollars)}';
    return formatter.format(dollars);
  }

  String _currencySymbol() {
    switch (currency?.toLowerCase()) {
      case 'usd': return '\$';
      case 'eur': return '€';
      case 'gbp': return '£';
      case 'inr': return '₹';
      default: return currency?.toUpperCase() ?? '';
    }
  }

  // Status booleans
  bool get isCompleted => status?.toLowerCase() == 'succeeded';
  bool get isPending   => status?.toLowerCase() == 'pending';
  bool get isFailed    => status?.toLowerCase() == 'failed';
  bool get isCanceled  => status?.toLowerCase() == 'canceled';

  // Credit/Debit helpers
  bool get isCredit => kind == TransactionKind.refund ||
      (amount != null && amount! > 0 && kind != TransactionKind.charge);
  bool get isDebit  => kind == TransactionKind.charge ||
      kind == TransactionKind.received || (amount != null && amount! < 0);

  // Display strings
  String get displayTitle {
    switch (kind) {
      case TransactionKind.received: return 'Payment';
      case TransactionKind.refund:   return 'Refund';
      case TransactionKind.charge:   return 'Charge';
      case TransactionKind.other:    return 'Transaction';
    }
  }

  String get statusText {
    switch (status?.toLowerCase()) {
      case 'succeeded':          return 'Completed';
      case 'pending':            return 'Processing';
      case 'failed':             return 'Failed';
      case 'canceled':           return 'Cancelled';
      case 'refunded':           return 'Refunded';
      case 'partially_refunded': return 'Partially Refunded';
      default: return status?.replaceAll('_', ' ').toUpperCase() ?? 'Unknown';
    }
  }

  TransactionHistoryEntity copyWith({...}) => TransactionHistoryEntity(...);
}
```

---

## STEP 5 — DATASOURCE PATTERN

```dart
abstract class WalletTransactionHistoryDataSource {
  Future<Either<Failure, PaginatedTransactionHistoryModel>>
      getTransactionHistory(PaginationParams params);
}

class WalletTransactionHistoryDataSourceImpl
    implements WalletTransactionHistoryDataSource {
  final DioApiService apiService = locator<DioApiService>(); // ← locator always

  @override
  Future<Either<Failure, PaginatedTransactionHistoryModel>>
      getTransactionHistory(PaginationParams params) async {
    try {
      ApiResponse res = await apiService.get(
        ApiEndpoints.getTransactions,        // ← ADD endpoint to ApiEndpoints const
        queryParameters: {"page": params.page, "limit": params.limit},
      );
      return Right(PaginatedTransactionHistoryModel.fromJson(res.data));
    } on ApiException catch (e) {
      debugPrint("Error in getTransactionHistory: $e");
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      debugPrint("Unexpected error in getTransactionHistory: $e");
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }
}
```

---

## STEP 6 — REPOSITORY PATTERN

```dart
// Domain interface
abstract class WalletTransactionHistoryRepository {
  Future<Either<Failure, PaginatedTransactionHistoryEntity>>
      getTransactionHistory({required PaginationParams params});
}

// Data impl
class WalletTransactionHistoryRepoImpl
    implements WalletTransactionHistoryRepository {
  final WalletTransactionHistoryDataSource dataSource;
  const WalletTransactionHistoryRepoImpl(this.dataSource);

  @override
  Future<Either<Failure, PaginatedTransactionHistoryEntity>>
      getTransactionHistory({required PaginationParams params}) async {
    final response = await dataSource.getTransactionHistory(params);
    return response.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (res) => Right(res.toEntity()),      // ← always toEntity(), never manual
    );
  }
}
```

---

## STEP 7 — USECASE PATTERN

```dart
class GetTransactionHistoryUseCase
    implements UseCase<PaginatedTransactionHistoryEntity, PaginationParams> {
  final WalletTransactionHistoryRepository repository;
  GetTransactionHistoryUseCase(this.repository);

  @override
  Future<Either<Failure, PaginatedTransactionHistoryEntity>> call(
    PaginationParams params,
  ) async => await repository.getTransactionHistory(params: params);
}
```

---

## STEP 8 — EVENTS PATTERN

```dart
sealed class TransactionHistoryEvent extends Equatable {
  const TransactionHistoryEvent();
  @override
  List<Object?> get props => [];
}

final class FetchTransactionHistory extends TransactionHistoryEvent {
  final bool isInitial;
  final PaginationParams params;

  const FetchTransactionHistory({this.isInitial = false, required this.params});

  @override
  List<Object?> get props => [isInitial, params];
}

class ClearTransactionDataEvent extends TransactionHistoryEvent {}
```

---

## STEP 9 — STATES PATTERN

```dart
abstract class TransactionHistoryState {}

class TransactionHistoryInitial extends TransactionHistoryState {}
class TransactionHistoryLoading extends TransactionHistoryState {}
class TransactionHistoryForceLoading extends TransactionHistoryState {}

class TransactionHistoryLoaded extends TransactionHistoryState {
  final List<TransactionHistoryEntity> transactions;
  final bool hasMore;
  final bool isLoadingMore;

  TransactionHistoryLoaded({
    required this.transactions,
    this.hasMore = false,
    this.isLoadingMore = false,
  });

  TransactionHistoryLoaded copyWith({
    List<TransactionHistoryEntity>? transactions,
    bool? hasMore,
    bool? isLoadingMore,
  }) => TransactionHistoryLoaded(
    transactions: transactions ?? this.transactions,
    hasMore: hasMore ?? this.hasMore,
    isLoadingMore: isLoadingMore ?? this.isLoadingMore,
  );
}

class TransactionHistoryError extends TransactionHistoryState {
  final String message;
  TransactionHistoryError(this.message);
}
```

---

## STEP 10 — BLOC PATTERN (3 GUARDS)

```dart
class TransactionHistoryBloc
    extends Bloc<TransactionHistoryEvent, TransactionHistoryState> {
  final GetTransactionHistoryUseCase _useCase;

  bool _hasMore = true;
  bool _isFetching = false;
  final List<TransactionHistoryEntity> _transactions = [];

  TransactionHistoryBloc(this._useCase) : super(TransactionHistoryInitial()) {
    on<FetchTransactionHistory>(_fetchTransactions);
    on<ClearTransactionDataEvent>((event, emit) {
      emit(TransactionHistoryLoading());
      _transactions.clear();
      _hasMore = true;
      _isFetching = false;
    });
  }

  Future<void> _fetchTransactions(
    FetchTransactionHistory event,
    Emitter<TransactionHistoryState> emit,
  ) async {
    if (_isFetching) return;
    if (!_hasMore && !event.isInitial) return;

    _isFetching = true;

    if (event.isInitial) {
      _transactions.clear();
      _hasMore = true;
      _isFetching = false;
      emit(TransactionHistoryLoading());
    }

    final result = await _useCase(event.params);

    result.fold(
      (failure) => emit(TransactionHistoryError(failure.message)),
      (res) {
        if (event.isInitial) _transactions.clear();
        _transactions.addAll(res.transactionHistory);
        _hasMore = res.pagination.hasMore ?? false;
        emit(TransactionHistoryLoaded(
          transactions: List.from(_transactions),
          hasMore: _hasMore,
        ));
      },
    );

    _isFetching = false;
  }
}
```

---

## STEP 11 — FILTER ENUM + HELPER CLASS PATTERN

```dart
// In wallet_screen.dart (top of file)
enum TransactionFilter { all, credit, debit, today, week, month }

extension on TransactionFilter {
  String get label {
    switch (this) {
      case TransactionFilter.all:   return 'All';
      case TransactionFilter.credit: return 'Credit';
      case TransactionFilter.debit:  return 'Debit';
      case TransactionFilter.today:  return 'Today';
      case TransactionFilter.week:   return 'This Week';
      case TransactionFilter.month:  return 'This Month';
    }
  }
}

// Static helper class — filter logic NOT in bloc
class TransactionFilterHelper {
  static List<TransactionHistoryEntity> apply(
    List<TransactionHistoryEntity> list,
    TransactionFilter filter,
  ) {
    final now = DateTime.now();
    switch (filter) {
      case TransactionFilter.all:    return list;
      case TransactionFilter.credit: return list.where((t) => t.kind == TransactionKind.received).toList();
      case TransactionFilter.debit:  return list.where((t) => t.kind == TransactionKind.charge).toList();
      case TransactionFilter.today:
        return list.where((t) {
          final d = t.createdAt.toLocal();
          return d.year == now.year && d.month == now.month && d.day == now.day;
        }).toList();
      case TransactionFilter.week:
        final weekAgo = now.subtract(const Duration(days: 7));
        return list.where((t) => t.createdAt.toLocal().isAfter(weekAgo)).toList();
      case TransactionFilter.month:
        return list.where((t) =>
          t.createdAt.toLocal().month == now.month &&
          t.createdAt.toLocal().year == now.year,
        ).toList();
    }
  }
}
```

---

## STEP 12 — SCREEN STRUCTURE PATTERN

Screen is split into **private widget classes** inside same file. No separate widget files.

```dart
// State holds filter + scroll
class _WalletScreenState extends State<WalletScreen> {
  TransactionFilter selectedFilter = TransactionFilter.all;
  final ScrollController _scrollController = ScrollController();
  int page = 1;

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  void _onScroll() {
    final bloc = context.read<TransactionHistoryBloc>();
    final state = bloc.state;
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      if (state is TransactionHistoryLoaded && state.hasMore) {
        page++;
        bloc.add(FetchTransactionHistory(
          params: PaginationParams(page: page, limit: 10),
        ));
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return GradientScaffold(
      appBar: AppBarComponent(title: "Wallet", centerTitle: true),
      body: Column(children: [
        _BalanceCard(),        // ← shows current balance
        Expanded(child: _TransactionsSection(
          scrollController: _scrollController,
          selectedFilter: selectedFilter,
          onFilterTap: _showFilterBottomSheet,
          onRefresh: _refreshTransactions,
        )),
      ]),
    );
  }

  Future<void> _refreshTransactions() async {
    page = 1;
    context.read<TransactionHistoryBloc>().add(FetchTransactionHistory(
      isInitial: true,
      params: PaginationParams(page: 1, limit: 10),
    ));
  }

  void _showFilterBottomSheet() {
    showModalBottomSheet(
      context: context,
      backgroundColor: Colors.transparent,
      isScrollControlled: true,
      builder: (_) => _FilterBottomSheet(
        selectedFilter: selectedFilter,
        onSelect: (filter) {
          setState(() => selectedFilter = filter);
          Navigator.pop(context);
        },
      ),
    );
  }
}
```

---

## STEP 13 — DIALOG/BOTTOM SHEET FACTORY PATTERN

```dart
// Bottom sheet with drag handle
class _FilterBottomSheet extends StatelessWidget {
  final TransactionFilter selectedFilter;
  final ValueChanged<TransactionFilter> onSelect;

  const _FilterBottomSheet({required this.selectedFilter, required this.onSelect});

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Container(
        decoration: BoxDecoration(
          color: AppColors.white,
          borderRadius: BorderRadius.vertical(top: Radius.circular(24.r)),
        ),
        child: Column(mainAxisSize: MainAxisSize.min, children: [
          // ── Drag handle
          Container(
            margin: EdgeInsets.symmetric(vertical: 12.h),
            width: 40.w, height: 4.h,
            decoration: BoxDecoration(
              color: Colors.grey.shade300,
              borderRadius: BorderRadius.circular(2.r),
            ),
          ),
          Padding(
            padding: EdgeInsets.all(16.w),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                TextComponent(text: 'Filter Transactions', fontsize: 18.sp, fontWeight: FontWeight.w600),
                VerticalSpacing(16),
                ...TransactionFilter.values.map((f) => _FilterOption(
                  filter: f, isSelected: f == selectedFilter,
                  onTap: () => onSelect(f),
                )),
                VerticalSpacing(16),
              ],
            ),
          ),
        ]),
      ),
    );
  }
}

// ── Reusable filter option tile
class _FilterOption extends StatelessWidget {
  final TransactionFilter filter;
  final bool isSelected;
  final VoidCallback onTap;
  const _FilterOption({required this.filter, required this.isSelected, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        margin: EdgeInsets.only(bottom: 8.h),
        padding: EdgeInsets.symmetric(horizontal: 16.w, vertical: 12.h),
        decoration: BoxDecoration(
          color: isSelected ? AppColors.primary : Colors.transparent,
          borderRadius: BorderRadius.circular(8.r),
          border: Border.all(color: isSelected ? AppColors.primary : Colors.grey.shade300),
        ),
        child: Row(children: [
          Icon(Icons.filter_list, size: 20.sp, color: isSelected ? AppColors.white : Colors.grey.shade600),
          HorizontalSpacing(12),
          TextComponent(
            text: filter.label, fontsize: 14.sp,
            fontWeight: isSelected ? FontWeight.w600 : FontWeight.w400,
            color: isSelected ? AppColors.white : AppColors.black,
          ),
        ]),
      ),
    );
  }
}
```

---

## STEP 14 — BLOCBUILDER / BLOCCONSUMER PATTERN

```dart
BlocConsumer<TransactionHistoryBloc, TransactionHistoryState>(
  listener: (context, state) {
    if (state is TransactionHistoryError) {
      AwesomeNotifier.showSnackBar(
        context: context,
        title: 'Oops!',
        message: state.message,
        contentType: ContentType.failure,
      );
    }
  },
  builder: (context, state) {
    if (state is TransactionHistoryLoading) return _buildSkeletonLoader();
    if (state is TransactionHistoryLoaded) {
      final filteredList = TransactionFilterHelper.apply(state.transactions, selectedFilter);
      return RefreshIndicator(
        onRefresh: onRefresh,
        child: filteredList.isEmpty
            ? _buildEmptyState()
            : ListView.builder(
                controller: scrollController,
                padding: EdgeInsets.symmetric(horizontal: 16.w),
                physics: const AlwaysScrollableScrollPhysics(),
                itemCount: filteredList.length + (state.hasMore ? 1 : 0),
                itemBuilder: (context, index) {
                  if (index < filteredList.length) {
                    return _buildTransactionItem(filteredList[index]);
                  }
                  return const Padding(
                    padding: EdgeInsets.all(16),
                    child: Center(child: CircularProgressIndicator()),
                  );
                },
              ),
      );
    }
    return const SizedBox.shrink();
  },
)
```

---

## STEP 15 — SKELETONIZER LOADING PATTERN

```dart
Widget _buildSkeletonLoader({int itemCount = 5}) {
  return Skeletonizer(
    child: ListView.builder(
      itemCount: itemCount,
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      padding: EdgeInsets.symmetric(horizontal: 16.sp, vertical: 10),
      itemBuilder: (context, index) => _buildTransactionItem(
        TransactionHistoryEntity(    // ← mock entity for skeleton shape
          id: "$index", user: "user$index",
          eventId: "event$index", eventType: "type$index",
          kind: TransactionKind.charge,
          createdAt: DateTime.now(), updatedAt: DateTime.now(),
        ),
      ),
    ),
  );
}
```

---

## STEP 16 — EMPTY STATE PATTERN

```dart
Widget _buildEmptyState() => ListView(children: [
  SizedBox(height: 100.h),
  Icon(Icons.receipt_long_outlined, size: 64.sp, color: Colors.grey.shade400),
  VerticalSpacing(16),
  TextComponent(
    text: 'No transactions found', fontsize: 16.sp,
    fontWeight: FontWeight.w500, color: Colors.grey.shade600,
    textAlign: TextAlign.center,
  ),
  VerticalSpacing(8),
  TextComponent(
    text: 'Your transaction history will appear here', fontsize: 14.sp,
    fontWeight: FontWeight.w400, color: Colors.grey.shade500,
    textAlign: TextAlign.center,
  ),
]);
```

---

## STEP 17 — DI REGISTRATION (service_locator.dart)

Add in order: DataSource → Repository → UseCase → Bloc

```dart
// DataSource
locator.registerLazySingleton<WalletTransactionHistoryDataSource>(
  () => WalletTransactionHistoryDataSourceImpl(),
);

// Repository
locator.registerLazySingleton<WalletTransactionHistoryRepository>(
  () => WalletTransactionHistoryRepoImpl(locator<WalletTransactionHistoryDataSource>()),
);

// UseCase
locator.registerLazySingleton(
  () => GetTransactionHistoryUseCase(locator<WalletTransactionHistoryRepository>()),
);

// Bloc — registerFactory (not singleton — fresh instance per page)
locator.registerFactory(
  () => TransactionHistoryBloc(locator<GetTransactionHistoryUseCase>()),
);
```

---

## STEP 18 — ROUTE ENUM ENTRY

```dart
// In lib/core/navigation/route_enums.dart
enum Routes {
  // ... existing routes ...
  walletScreen('/wallet/wallet_screen'),
  // ...
  final String path;
  const Routes(this.path);
}
```

---

## STEP 19 — GOROUTER REGISTRATION (app_router.dart)

```dart
GoRoute(
  path: Routes.walletScreen.path,
  pageBuilder: (context, state) => AppRouter._buildPage(
    BlocProvider(
      create: (_) => locator<TransactionHistoryBloc>()
        ..add(FetchTransactionHistory(
          isInitial: true,
          params: PaginationParams(page: 1, limit: 10),
        )),
      child: const WalletScreen(),
    ),
    state,
  ),
),
```

**Navigation to wallet screen:**
```dart
context.go(Routes.walletScreen.path);
// OR
context.push(Routes.walletScreen.path);
```

---

## STEP 20 — API ENDPOINT ADDITION

```dart
// In lib/core/constants/api_endpoints.dart
class ApiEndpoints {
  // ... existing endpoints ...
  static const String getTransactions = '/wallet/transactions'; // ← from user
}
```

---

## TRANSACTION ITEM CARD PATTERN

```dart
Widget _buildTransactionItem(TransactionHistoryEntity transaction) {
  final isCredit = transaction.kind == TransactionKind.received;
  final isRefund  = transaction.kind == TransactionKind.refund;
  final isCharge  = transaction.kind == TransactionKind.charge;

  return Container(
    margin: EdgeInsets.only(bottom: 12.h),
    padding: EdgeInsets.all(16.w),
    decoration: BoxDecoration(
      color: Colors.grey.shade50,
      borderRadius: BorderRadius.circular(12.r),
      border: Border.all(color: Colors.grey.shade200),
    ),
    child: Row(children: [
      // ── Icon container
      Container(
        width: 48.w, height: 48.h,
        decoration: BoxDecoration(
          color: isCredit ? Colors.green.shade50 : Colors.red.shade50,
          borderRadius: BorderRadius.circular(12.r),
        ),
        child: Center(child: SvgPicture.asset(ImageConstants.appLogo)),
      ),
      HorizontalSpacing(12),
      // ── Title + Date
      Expanded(child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          TextComponent(text: transaction.displayTitle, fontsize: 14.sp, fontWeight: FontWeight.w600),
          VerticalSpacing(2),
          TextComponent(
            text: DateFormat('dd MMM yyyy, hh:mm a').format(transaction.createdAt.toLocal()),
            fontsize: 11.sp, fontWeight: FontWeight.w400, color: Colors.grey.shade500,
          ),
        ],
      )),
      // ── Amount
      TextComponent(
        text: '${(isCredit || isRefund) ? '+' : ''}${transaction.getFormattedAmount(withCurrency: true)}',
        fontsize: 16.sp, fontWeight: FontWeight.w700,
        color: (isCredit || isRefund) ? Colors.green.shade600
               : isCharge ? Colors.red.shade600 : Colors.black87,
      ),
    ]),
  );
}
```

---

## BALANCE CARD PATTERN

```dart
class _BalanceCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.symmetric(horizontal: 16.w),
      child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
        VerticalSpacing(17),
        Container(
          width: double.infinity,
          decoration: BoxDecoration(
            color: AppColors.white,
            borderRadius: BorderRadius.circular(20.r),
          ),
          padding: EdgeInsets.all(20.w),
          child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
            TextComponent(text: 'Available Balance', fontsize: 16.sp, fontWeight: FontWeight.w400),
            VerticalSpacing(5),
            TextComponent(
              text: '\$490.00',           // ← replace with real balance API
              fontsize: 32.sp, fontWeight: FontWeight.w700,
              color: AppColors.appGradient,  // gradient color support
            ),
          ]),
        ),
        VerticalSpacing(15),
      ]),
    );
  }
}
```

---

## SHARED WIDGETS USED

| Widget | Import | Usage |
|---|---|---|
| `GradientScaffold` | `shared/widgets/gradient_scaffold.dart` | Root scaffold |
| `AppBarComponent` | `shared/widgets/app_bar_component.dart` | `AppBarComponent(title: "Wallet", centerTitle: true)` |
| `TextComponent` | `shared/widgets/text_component.dart` | All text rendering |
| `VerticalSpacing(n)` | `core/utils/spacing.dart` | Vertical gaps |
| `HorizontalSpacing(n)` | `core/utils/spacing.dart` | Horizontal gaps |
| `AwesomeNotifier.showSnackBar` | `core/utils/awesome_notifier.dart` | Error/success toasts |

---

## SIZING RULES

- All sizes via `flutter_screenutil`: `.w` width, `.h` height, `.sp` font, `.r` radius
- Padding horizontal: `16.w`
- Card radius: `12.r` to `20.r`
- Card padding: `16.w` or `20.w`
- List item margin bottom: `12.h`
- Icon size: `16.sp` to `64.sp`

---

## DEPENDENCIES CHECKLIST

```yaml
dependencies:
  flutter_bloc: ^8.x
  bloc: ^8.x
  dartz: ^0.10.x
  get_it: ^7.x
  go_router: ^13.x
  flutter_screenutil: ^5.x
  skeletonizer: ^1.x
  awesome_snackbar_content: ^0.1.x
  intl: ^0.19.x
  equatable: ^2.x
  flutter_svg: ^2.x
```

---

## GENERATION CHECKLIST

When building this feature, complete in order:

- [ ] Ask user for: endpoint path, query params, response shape, item field names
- [ ] Add endpoint to `ApiEndpoints`
- [ ] Create TransactionKind enum + extension (or custom enum from user's types)
- [ ] Create `TransactionHistoryModel` (fromJson/toJson/toEntity/fromEntity)
- [ ] Create `PaginatedTransactionHistoryModel`
- [ ] Create `TransactionHistoryEntity` with business helpers
- [ ] Create `PaginatedTransactionHistoryEntity`
- [ ] Create `WalletTransactionHistoryDataSource` (abstract + impl)
- [ ] Create `WalletTransactionHistoryRepository` (abstract + impl)
- [ ] Create `GetTransactionHistoryUseCase`
- [ ] Create events (Fetch + Clear)
- [ ] Create states (Initial/Loading/Loaded/Error/ForceLoading)
- [ ] Create bloc with 3 guards
- [ ] Create `TransactionFilter` enum + `TransactionFilterHelper` class
- [ ] Create screen with private classes (_BalanceCard, _TransactionsSection, _FilterBottomSheet, _FilterOption)
- [ ] Add DI to service_locator.dart (DataSource → Repo → UseCase → Bloc)
- [ ] Add route to Routes enum
- [ ] Add GoRoute to app_router.dart
- [ ] Verify: imports, package name, no missing widgets

**Why:** Extracted from production cier_check_user app. This is the exact architecture, naming, and pattern used in the live wallet feature.
**How to apply:** Follow all 20 steps in order. Ask user ONLY for API details. Generate everything else from this skill.
