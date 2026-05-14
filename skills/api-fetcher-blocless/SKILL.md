---
name: api-fetcher-blocless
description: Generate complete API integration code following clean architecture (Data → Domain → Presentation) without Bloc layer. Perfect for Flutter projects using service locator pattern and Either<Failure, T> error handling.
tags: [architecture, api, flutter, clean-code, code-generation, datasource, repository, usecase, entity]
scope: global
---

# API Fetcher Blocless

A professional code generation skill that scaffolds API integration across the three-layer clean architecture: **Data Layer**, **Domain Layer**, and **Presentation Layer**—without Bloc dependencies. Perfect for rapidly implementing new API endpoints while maintaining architectural consistency.

## When to Use This Skill

- ✅ Creating new API features with clean architecture
- ✅ Implementing multiple datasources, models, and repositories
- ✅ Setting up service locator dependency injection
- ✅ Building usecases that wrap business logic
- ✅ Generating boilerplate code for API endpoints
- ✅ Maintaining consistent error handling with Either<Failure, T>

## Architecture Pattern

This skill generates code following a proven clean architecture structure:

```
Feature (e.g., auth)
├── data/
│   ├── datasources/         → API calls & error handling
│   ├── models/              → JSON serialization/deserialization
│   └── repositories/        → Implementation, bridges Data ↔ Domain
├── domain/
│   ├── entities/            → Pure data classes
│   ├── repositories/        → Abstract contracts
│   ├── usecases/            → Business logic wrappers
│   └── params/              → Request parameters
└── presentation/
    └── screens/             → UI layer (no Bloc)
```

## Core Workflow

### 1. **Identify Requirements**
- API endpoint URL
- Request parameters (headers, body, query)
- Response data structure
- Feature name (lowercase, snake_case)

### 2. **Data Layer Implementation**

#### Step 2.1: Create Params (if needed)
```dart
// core/params/[feature]_params.dart
class [Feature]Params {
  final String param1;
  final int param2;
  
  const [Feature]Params({
    required this.param1,
    required this.param2,
  });
}
```

#### Step 2.2: Create Entity
```dart
// domain/entities/[feature]_entity.dart
class [Feature]Entity {
  final String field1;
  final int field2;
  
  const [Feature]Entity({
    required this.field1,
    required this.field2,
  });
}
```

#### Step 2.3: Create Model (extends Entity)
```dart
// data/models/[feature]_model.dart
class [Feature]Model extends [Feature]Entity {
  const [Feature]Model({
    required super.field1,
    required super.field2,
  });

  factory [Feature]Model.fromJson(Map<String, dynamic> json) {
    return [Feature]Model(
      field1: json['field1']?.toString() ?? '',
      field2: json['field2'] as int? ?? 0,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'field1': field1,
      'field2': field2,
    };
  }
}
```

#### Step 2.4: Create Abstract Datasource
```dart
// data/datasources/[feature]_datasource.dart
abstract class [Feature]Datasource {
  Future<Either<Failure, [Feature]Model>> [methodName]([Feature]Params params);
}
```

#### Step 2.5: Implement Datasource
```dart
// data/datasources/[feature]_datasource.dart
class [Feature]DatasourceImpl implements [Feature]Datasource {
  final DioApiService apiService = locator<DioApiService>();

  @override
  Future<Either<Failure, [Feature]Model>> [methodName]([Feature]Params params) async {
    try {
      ApiResponse res = await apiService.post(
        ApiEndpoints.[method],
        data: params,
        config: RequestConfig(headers: {'header': 'value'}),
      );
      [Feature]Model data = [Feature]Model.fromJson(res.data);
      return Right(data);
    } on ApiException catch (e) {
      debugPrint("[Feature] error: $e");
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      debugPrint("[Feature] unexpected error: $e");
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }
}
```

#### Step 2.6: Create Abstract Repository
```dart
// domain/repositories/[feature]_repository.dart
import 'package:dartz/dartz.dart';

abstract class [Feature]Repository {
  Future<Either<Failure, [Feature]Entity>> [methodName]([Feature]Params params);
}
```

#### Step 2.7: Implement Repository
```dart
// data/repositories/[feature]_repository_impl.dart
class [Feature]RepositoryImpl implements [Feature]Repository {
  final [Feature]Datasource dataSource;

  const [Feature]RepositoryImpl(this.dataSource);

  @override
  Future<Either<Failure, [Feature]Entity>> [methodName]([Feature]Params params) async {
    final response = await dataSource.[methodName](params);
    
    return response.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (right) => Right(right),
    );
  }
}
```

### 3. **Domain Layer Implementation**

#### Step 3.1: Create Usecase
```dart
// domain/usecases/[feature]_usecase.dart
class [Feature]Usecase {
  final [Feature]Repository repository;

  [Feature]Usecase(this.repository);

  Future<Either<Failure, [Feature]Entity>> call([Feature]Params params) async {
    return await repository.[methodName](params);
  }
}
```

### 4. **Presentation Layer Integration**

#### Step 4.1: Inject Dependencies
```dart
// core/services/service_locator.dart (add to setupServiceLocator)
locator.registerSingleton<[Feature]Datasource>([Feature]DatasourceImpl());
locator.registerSingleton<[Feature]Repository>(
  [Feature]RepositoryImpl(locator<[Feature]Datasource>()),
);
locator.registerSingleton<[Feature]Usecase>(
  [Feature]Usecase(locator<[Feature]Repository>()),
);
```

#### Step 4.2: Use in Presentation
```dart
// presentation/screens/[feature]_screen.dart
class [Feature]Screen extends StatelessWidget {
  final usecase = locator<[Feature]Usecase>();

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        final result = await usecase([Feature]Params(
          param1: 'value1',
          param2: 123,
        ));
        
        result.fold(
          (failure) => ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text(failure.message)),
          ),
          (data) => print('Success: $data'),
        );
      },
      child: const Text('Call API'),
    );
  }
}
```

## Error Handling Pattern

Always follow the Either<Failure, SuccessType> pattern:

```dart
// ✅ Correct: Handles both error and success paths
return response.fold(
  (failure) => Left(ServerFailure(message: failure.message)),
  (data) => Right(data),
);

// ❌ Wrong: Missing error handling
return Right(response);
```

## Key Principles

1. **Single Responsibility**: Each file has one purpose
2. **Immutability**: All entities and models are immutable (use `const`)
3. **Type Safety**: Use `Either<Failure, T>` for error handling
4. **DI Pattern**: Always inject dependencies via constructor
5. **Null Safety**: Handle null values in JSON deserialization
6. **Consistent Naming**: Use feature_name in snake_case for files

## Import Pattern

Every implementation file should import:
```dart
import 'package:dartz/dartz.dart';                    // Either, Right, Left
import 'package:your_app/core/errors/...';           // Failure types
import 'package:your_app/core/network/...';          // API service
import 'package:your_app/core/params/...';           // Params
import 'package:your_app/core/services/...';         // Service locator
import 'package:flutter/foundation.dart';             // debugPrint
```

## Quality Checklist

Before generating code, ensure:
- [ ] Feature name is snake_case
- [ ] All files follow the layer structure
- [ ] Params class has constructor with named parameters
- [ ] Model `fromJson` handles null values safely
- [ ] Repository uses `fold()` for error handling
- [ ] Datasource catches both `ApiException` and generic exceptions
- [ ] Usecase simply wraps repository call
- [ ] Dependencies registered in service_locator
- [ ] No direct API calls in presentation layer
- [ ] No Bloc-related imports in generated code

## Example Usage Prompts

**Prompt 1 - Quick API Integration:**
```
API Fetcher Blocless: Generate code for payments
- Endpoint: POST /api/payments
- Request params: userId (String), amount (double)
- Response: {success: bool, transactionId: String, status: String}
```

**Prompt 2 - Full Feature Implementation:**
```
Using API Fetcher Blocless skill, create complete implementation for:
Feature: wallet
API: GET /api/wallet/balance
Response: {balance: double, currency: String, lastUpdated: String}
```

**Prompt 3 - Multiple Methods:**
```
Generate API Fetcher Blocless code for notifications feature with:
1. getNotifications(userId) → List<NotificationEntity>
2. markAsRead(notificationId) → bool
```

## Common Pitfalls ❌

- Putting business logic in datasource
- Forgetting to handle null in `fromJson()`
- Using mutable data structures (List, Map without const)
- Skipping the usecase layer
- Direct API calls from UI
- Mixing Bloc with this architecture
- Not using Either for error handling
- Forgetting to register in service_locator

## Pro Tips 💡

1. **Always validate JSON**: Use `?.toString()` and `?? defaultValue` for null safety
2. **Use debugPrint**: Helps debug API calls in production builds
3. **Consistent naming**: [Feature] should match feature folder name (pascalCase → snake_case)
4. **DRY principle**: Share common request/response patterns in base classes
5. **Test early**: Write unit tests for datasources before integration
6. **Document endpoints**: Add comments showing API docs URL in datasource

## Real-World Example: Auth Feature

### Directory Structure
```
lib/features/auth/
├── data/
│   ├── datasources/auth_datasource.dart
│   ├── models/social_signin_model.dart
│   └── repositories/auth_repository_impl.dart
├── domain/
│   ├── entities/social_signin_entity.dart
│   ├── repositories/google_signin_repository.dart
│   ├── repositories/apple_signin_repository.dart
│   ├── usecases/google_signin_usecase.dart
│   └── usecases/apple_signin_usecase.dart
└── presentation/
    └── screens/login_screen.dart
```

### Implementation Flow
1. **API call** → DioApiService.post() in Datasource
2. **Model conversion** → fromJson() creates SocialSigninModel
3. **Error handling** → Either<Failure, Model> returned
4. **Repository** → fold() handles both success/failure
5. **Usecase** → Wraps repository for business logic
6. **UI** → Calls usecase and displays result

## Related Skills & Next Steps

- Create a **Screen Generator** skill for building UI screens that consume these usecases
- Add an **Error Handler** skill for centralized failure handling
- Build a **State Management** skill (GetX, Provider) that integrates with this pattern

---

**Version**: 1.0  
**Last Updated**: May 6, 2026  
**Audience**: Flutter/Dart developers using Clean Architecture  
**Status**: ✅ Professional | Production-Ready | Globally Available