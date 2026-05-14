# Folder Structure

## Top-level `lib`
- `core`: shared systems and infrastructure.
- `features`: app features grouped by domain.
- `shared`: UI widgets and shared model classes.
- `main.dart` and flavor files (`main_dev.dart`, etc.).

## `core` breakdown
- `api_service`: `network.dart`, `interceptor.dart`, `api_response.dart`, `api_handler.dart`.
- `constants`: app-level constants and endpoints.
- `config`: environment configs.
- `di`: service locator.
- `errors`: exceptions and failure classes.
- `session`: session state and cubit.
- `services`: local storage and integrations.
- `theme`: theming.
- `routing`: app routes.
- `helpers`: utility helpers.

## `features` breakdown
Each feature usually includes:
- `data`: datasources, models, repositories.
- `domain`: entities, repositories, usecases, params.
- `presentation`: cubit/bloc, screens, widgets.
- `di` for feature injection.
- `routes` and `navigation` if needed.

## `shared`
- `widgets`: reusable UI components.
- `models`: app-level shared models.

## Additional fold
- `.agents`: AI assistant config and documentation.
