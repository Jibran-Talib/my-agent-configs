# Screen Template

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:tailored/shared/widgets/button_component.dart';
import 'package:tailored/shared/widgets/text_component.dart';

import '../../presentation/cubit/<feature>_cubit.dart';

class <Feature>Screen extends StatelessWidget {
  const <Feature>Screen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('<Feature>')),
      body: BlocConsumer<<Feature>Cubit, <Feature>State>(
        listener: (context, state) {
          if (state.status == <Feature>Status.error) {
            // show snackbar / response widget
          }
        },
        builder: (context, state) {
          if (state.status == <Feature>Status.loading) {
            return const Center(child: CircularProgressIndicator());
          }

          return Column(
            children: [
              TextComponent(text: 'Feature screen'),
              ButtonComponent.primary(
                text: 'Do action',
                onPressed: () =>
                    context.read<<Feature>Cubit>().doSomething(<params>),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

- Use `go_router` configured route names.
- Use theme text via `Theme.of(context)` or `context.textTheme`.
