# API Integration Workflow

1. Confirm endpoint path and payload contract with backend.

2. Add endpoint constant to `core/constants/api_endpoints.dart`.

3. Update domain request params in `features/<feature>/domain/usecases/*_params.dart`.

4. Implement remote datasource method in `features/<feature>/data/datasources`:
   - Use `apiService.request(endpoint: EndPoints.<name>, method: DioMethod.<verb>, data: params.toJson())`.
   - Check success and data existence.
   - Throw `Exception` with message on `!success`.

5. Update model in `features/<feature>/data/models` with:
   - `fromJson`, `toJson`, `fromEntity`, `toEntity`.

6. Add repository interface method in `features/<feature>/domain/repositories`.

7. Implement repository in `features/<feature>/data/repositories`:
   - Wrap with `executeApiRequest(() async { ... })`.
   - Return model.toEntity() or needed result.

8. Add usecase in `features/<feature>/domain/usecases` that calls old repository.

9. Update cubit/bloc in `features/<feature>/presentation`:
   - Set loading state.
   - Call usecase and fold on result.
   - Emit success or error states using message and payload.

10. Connect with UI screen and widget event for user interaction.

11. Update DI (`<feature>_injection.dart`) and routes (
`<feature>_routes.dart`) if new screens are added.

12. Add tests for datasource, repository, usecase, and cubit/bloc as possible.
