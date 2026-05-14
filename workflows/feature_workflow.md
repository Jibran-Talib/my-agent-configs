# Feature Development Workflow

1. Identify feature scope and API endpoints.

2. Add endpoint constant in `core/constants/api_endpoints.dart` (if needed).

3. Implement domain entities in `features/<feature>/domain/entities`.

4. Add repository interface in `features/<feature>/domain/repositories`.

5. Create usecase in `features/<feature>/domain/usecases`.
   - For parameters, create classes in `features/<feature>/domain/usecases/..._params.dart`.
   - Method signature: `Future<Either<Failure, T>> call(Params params)` or no params.

6. Implement remote datasource in `features/<feature>/data/datasources`.
   - Use `ApiService.request` with `DioMethod` and route from `EndPoints`.
   - Map `response.data` to model.

7. Implement repository in `features/<feature>/data/repositories`.
   - Use `executeApiRequest` to wrap remote datasource calls.
   - Convert models to entities.

8. Create presentation layer:
   - `cubit` or `bloc` in `features/<feature>/presentation`.
   - States with `status` enums and robust copying.
   - UI screens in `features/<feature>/presentation/screens`.

9. Add DI wiring in `features/<feature>/<feature>_injection.dart`.
   - Register datasource, repository, usecases, cubit/bloc.
   - Ensure feature injection called from `core/di/service_locator.dart`.

10. Add routes in `features/<feature>/<feature>_routes.dart` and include in `AppRouter`.

11. Add tests (unit for usecase/repository, widget for screen) if available.
