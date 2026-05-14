# Repository Template

```dart
import 'package:fpdart/fpdart.dart';
import '../../../../core/api_service/api_handler.dart';
import '../../../../core/errors/failure.dart';
import '../../domain/entities/<feature>_entity.dart';
import '../../domain/repositories/<feature>_repository.dart';
import '../../domain/usecases/<params>.dart';
import '../datasources/<feature>_remote_datasource.dart';

class <Feature>RepositoryImpl implements <Feature>Repository {
  final <Feature>RemoteDataSource remoteDataSource;

  <Feature>RepositoryImpl(this.remoteDataSource);

  @override
  Future<Either<Failure, <Feature>Entity>> get<Feature>() {
    return executeApiRequest(() async {
      final model = await remoteDataSource.get<Feature>();
      return model.toEntity();
    });
  }

  // add more methods following the same pattern
}
```

- Use `executeApiRequest` from `core/api_service/api_handler.dart`.
- Convert model to entity in repository layer.
