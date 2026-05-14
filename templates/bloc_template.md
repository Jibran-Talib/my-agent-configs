# Bloc/Cubit Template

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

part '<feature>_state.dart';

class <Feature>Cubit extends Cubit<<Feature>State> {
  final <UseCase> _useCase;

  <Feature>Cubit(this._useCase) : super(const <Feature>State());

  Future<void> doSomething(<Params> params) async {
    emit(state.copyWith(status: <Feature>Status.loading, message: null));

    final result = await _useCase(params);

    result.fold(
      (failure) => emit(
        state.copyWith(
          status: <Feature>Status.error,
          message: failure.message,
        ),
      ),
      (data) => emit(
        state.copyWith(
          status: <Feature>Status.success,
          <dataField>: data,
        ),
      ),
    );
  }
}
```

`<feature>_state.dart` should include status enum, state class with `copyWith`, and props.
