# AppDialog Rules

This project uses a centralized `AppDialog` class for all confirmation dialogs.
Do NOT use Flutter's `showDialog()` directly. Always use `AppDialog` static methods.

---

## Location

```
lib/shared/widgets/dialogs_widget/app_dialog.dart
lib/shared/widgets/dialogs_widget/dialog_types.dart
```

---

## Core Rules

- Never call `showDialog()` directly
- Never create custom one-off dialog widgets in screens
- Always use `AppDialog.<method>()` static methods
- Always call `AppDialog.close()` to dismiss — never `Navigator.pop()` inside dialogs
- Dialog logic must stay in UI layer — never call AppDialog from Bloc/Repository
- For new dialog types, add a new static method to `AppDialog` class

---

## Available Methods

### Confirmation Dialogs (void — fire and forget)

```dart
// Delete anything
AppDialog.delete(
  title: "Delete Trip",
  message: "Are you sure you want to delete this trip?",
  onConfirm: () => bloc.add(DeleteTripEvent(id)),
);

// Leave group
AppDialog.leave(
  onConfirm: () => bloc.add(LeaveGroupEvent(roomId: id)),
);

// Logout
AppDialog.logout(
  onConfirm: () {
    context.read<AuthCubit>().logout();
  },
);

// Remove member from group
AppDialog.removeMember(
  onConfirm: () => bloc.add(RemoveMemberEvent(userId: id)),
);

// Block user
AppDialog.blockUser(
  onConfirm: () => bloc.add(ToggleBlockEvent(userId: id, toggle: true)),
);

// Mute notifications
AppDialog.mute(
  onConfirm: () => bloc.add(ToggleMuteEvent(roomId: id, isMuted: true)),
);

// Clear chat
AppDialog.clearChat(
  onConfirm: () => bloc.add(ClearChatEvent()),
);

// Accept invitation
AppDialog.acceptInvitation(
  onConfirm: () => bloc.add(RespondInviteEvent(id: id, action: 'accept')),
);

// Reject invitation
AppDialog.rejectInvitation(
  onConfirm: () => bloc.add(RespondInviteEvent(id: id, action: 'reject')),
);

// Regenerate avatar
AppDialog.regenerate(
  onConfirm: () => cubit.regenerateAvatar(),
);

// Try on outfit
AppDialog.tryOn(
  onConfirm: () => cubit.tryOnAvatar(params: params),
);

// Warning / skip
AppDialog.warning(
  onConfirm: () => context.push(NextRoute.path),
);

// Subscription prompt
AppDialog.subscription(
  onConfirm: () => InAppPurchaseNavigation.toShop(context),
);

// LifeLike plan prompt
AppDialog.buyLifeLikePlan(
  onConfirm: () => InAppPurchaseNavigation.toShop(context, getAvatarPlan: true),
);
```

### Info Dialogs (no onConfirm needed)

```dart
// Success message
AppDialog.success();

// Upload instruction
AppDialog.instruction(onConfirm: () {});
```

### Dialog With Text Input

```dart
// Report user — onConfirm receives the typed reason
AppDialog.reportUser(
  onConfirm: (reason) {
    bloc.add(ReportUserEvent(ReportUserParams(userId: id, reason: reason)));
  },
);
```

### Async Dialog (returns a value)

```dart
// Avatar generation exit — returns bool
final shouldGoBack = await AppDialog.avatarGenerationExitWarning();
if (shouldGoBack == true) {
  context.pop();
}
```

### Custom Dialog

```dart
// Show any custom widget as dialog
AppDialog.show(
  MyCustomDialog(),
  barrierDismissible: false,
);
```

---

## isDanger Flag

Use `isDanger: true` when the confirm button should be red (destructive actions):

```dart
AppDialog.delete(
  isDanger: true,   // confirm button → red
  onConfirm: () => bloc.add(DeleteEvent()),
);

AppDialog.rejectInvitation(
  isDanger: true,   // confirm button → red
  onConfirm: () {},
);
```

Default `isDanger: false` → confirm button uses primary gradient.

---

## Adding a New Dialog

When a new confirmation dialog is needed, add a static method to `AppDialog`:

```dart
static void myNewDialog({
  required VoidCallback onConfirm,
  String title = "My Title",
  String message = "Are you sure?",
  String cancelText = "Cancel",
  String confirmText = "Confirm",
  bool isDanger = false,
}) {
  show(
    _DialogWidget(
      type: DialogType.warning, // choose appropriate type
      title: title,
      message: message,
      cancelText: cancelText,
      confirmText: confirmText,
      isDanger: isDanger,
      onConfirm: onConfirm,
    ),
  );
}
```

Then add the `DialogType` entry in `dialog_types.dart` if a new icon/color is needed.

---

## DO NOT

```dart
// ❌ Wrong — never use showDialog directly
showDialog(
  context: context,
  builder: (_) => AlertDialog(...),
);

// ❌ Wrong — never use CircularProgressIndicator inside dialogs
// ❌ Wrong — never call AppDialog from Bloc or Repository
// ❌ Wrong — never Navigator.pop() inside dialog buttons — use AppDialog.close()
// ❌ Wrong — never create one-off dialog widgets in screen files
```

---

## Correct Pattern

```dart
// ✅ Always use AppDialog static methods from UI layer
AppDialog.delete(
  title: "Delete Account",
  message: "This cannot be undone.",
  cancelText: "Keep Account",
  confirmText: "Delete Account",
  onConfirm: () {
    context.read<SettingsCubit>().deleteAccount(password);
  },
);
```
