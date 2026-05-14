# Loader Rules (Global Overlay Loader)

This project uses a global overlay loader system.
The loader is already initialized globally in main.dart using Overlay and NavigatorKey.
Do NOT create separate loaders in different screens.

Whenever loading is required, the global overlay loader must be shown.

---

# 1. Global Loader System

A global Loader is configured in main.dart and uses OverlayEntry.
This loader must be used everywhere in the app for loading states.

Use:
Loader.show()
Loader.hide()
Loader.during()

Do NOT use:

* CircularProgressIndicator directly
* Custom loaders per screen
* Dialog loaders
* Scaffold loaders
* Local overlay loaders

Always use the global overlay loader.

---

# 2. Loader Usage Rules

## Show Loader

Use when API call starts:
Loader.show();

## Hide Loader

Use when API call finishes:
Loader.hide();

## Preferred Method (Recommended)

Use Loader.during() for async operations:

await Loader.during(() async {
await apiCall();
});

This automatically shows and hides loader.

---

# 3. Bloc Loading Rule

Bloc should NOT control overlay loader directly.
UI layer should show loader based on loading state.

But for actions like:

* Submit form
* Update profile
* Upload images
* Delete item
* Create item
  Use Loader.during()

Example:
onPressed: () async {
await Loader.during(() async {
context.read<OrderBloc>().add(CreateOrderEvent());
});
}

---

# 4. Screen Loading vs Overlay Loading

There are two types of loading:

## Screen Loading

Used when loading entire screen data:
Use Loader widget wrapper.

Example:
Loader(
isLoading: state.isLoading,
child: Scaffold(...)
)

## Overlay Loading

Used when performing actions:

* Button click
* API submit
* Upload
* Delete
* Update
* Generate avatar
  Use:
  Loader.show()
  Loader.hide()
  OR
  Loader.during()

---

# 5. When to Use Overlay Loader

Overlay loader must be used for:

* API POST requests
* API PUT requests
* API DELETE requests
* Image upload
* File upload
* Form submit
* Generate avatar
* Payment processing
* Long operations
* Navigation waiting
* Any background API call

---

# 6. Important Rules

1. The loader is global and already initialized in main.dart.
2. Never create another loader system.
3. Never use CircularProgressIndicator directly.
4. Always use Loader.show() or Loader.during().
5. Always hide loader after operation.
6. Only one loader overlay can be visible at a time.
7. Loader must block user interaction.
8. Loader must appear above entire UI.
9. Loader logic must be handled in UI layer.
10. Repository and DataSource must not control loader.

---

# 7. Standard Loader Pattern Summary

## For API Actions:

await Loader.during(() async {
await repository.callApi();
});

## For Screen Loading:

Loader(
isLoading: state.isLoading,
child: ScreenUI()
)

This is the official loading system for this project.
All features must use this loader system.
