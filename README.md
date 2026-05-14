# Personal Flutter AI Agent Config

Personal rules, skills, workflows, and templates for Flutter development.
Use this config to make any AI coding tool behave like your personal senior Flutter developer.

---

## Repository Structure

```
.agents/
├── flutter_dev.agent.md          # Main agent definition (entry point)
│
├── rules/                        # Hard rules — agent must always follow
│   ├── coding_rules.md           # Clean architecture, API, DI, error handling
│   ├── folder_structure.md       # lib/ folder layout
│   ├── naming_conventions.md     # Files, classes, variables, enums
│   ├── pagination_rules.md       # Custom pagination via Bloc (not API response)
│   ├── loader_rules.md           # Global Loader.show/hide — no CircularProgressIndicator
│   ├── app_dialog_rules.md       # AppDialog static methods — no showDialog directly
│   └── skeletonizer_rules.md     # Skeletonizer for loading UI — no spinners
│
├── skills/                       # How to implement specific features
│   ├── project_architecture.md   # Clean arch overview, API flow, DI flow
│   ├── state_management.md       # Cubit/Bloc conventions, state patterns
│   ├── firebase_remote_config_skill.md   # Firebase Remote Config full implementation
│   ├── notification_service_skill.md     # FCM + Local Notifications full implementation
│   └── location_service_skill.md         # Geolocator + Google Maps Geocoding
│
├── workflows/                    # Step-by-step process guides
│   ├── feature_workflow.md       # How to build a complete feature end-to-end
│   └── api_integration_workflow.md  # How to integrate a new API endpoint
│
└── templates/                    # Ready-to-use code templates
    ├── bloc_template.md          # Bloc/Cubit + State boilerplate
    ├── repository_template.md    # Repository implementation boilerplate
    └── screen_template.md        # Screen with BlocConsumer boilerplate
```

---

## New Machine Setup

Clone this repo to your global agents folder:

```bash
git clone https://github.com/Jibran-Talib/my-agent-configs.git C:/Users/<your-username>/.agents
```

Then follow the setup for whichever AI tool you use below.

---

## GitHub Copilot Setup

### One-Time Global Setup (VS Code)

1. Open VS Code
2. Press `Ctrl + Shift + P` → type **"Open User Settings JSON"** → open it
3. Add this block inside the JSON:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "C:/Users/<your-username>/.agents/rules/coding_rules.md"
    },
    {
      "file": "C:/Users/<your-username>/.agents/rules/folder_structure.md"
    },
    {
      "file": "C:/Users/<your-username>/.agents/rules/naming_conventions.md"
    },
    {
      "file": "C:/Users/<your-username>/.agents/rules/pagination_rules.md"
    },
    {
      "file": "C:/Users/<your-username>/.agents/rules/loader_rules.md"
    },
    {
      "file": "C:/Users/<your-username>/.agents/rules/app_dialog_rules.md"
    },
    {
      "file": "C:/Users/<your-username>/.agents/rules/skeletonizer_rules.md"
    }
  ]
}
```

> Replace `<your-username>` with your actual Windows username.

4. Save the file — Copilot now follows all your rules globally.

### Every New Project Setup

Create this file in every new project root:

**`.github/copilot-instructions.md`**

```markdown
You are a senior Flutter developer working on this project.

Architecture: Clean Architecture with domain / data / presentation layers.
State management: Bloc and Cubit only (flutter_bloc package).
DI: GetIt via service_locator.dart — register as lazy singletons.
Network: Dio via ApiService with DioMethod enums and executeApiRequest helper.
Error handling: Either<Failure, T> in repositories, core failures in core/errors/failure.dart.

Rules:
- UseCase method: call()
- Repository class: FeatureRepositoryImpl implements FeatureRepository
- snake_case files, PascalCase classes
- Pagination: always use PaginationParams, control from Bloc, ignore API totalPages
- Loading UI: always use Skeletonizer — never CircularProgressIndicator
- Action loading: Loader.show() / Loader.hide() / Loader.during() — never create new loaders
- Dialogs: always use AppDialog static methods — never showDialog() directly
```

### How to Use Skills with Copilot

In Copilot Chat, reference the skill file directly:

```
@workspace use the pattern from C:/Users/<username>/.agents/skills/notification_service_skill.md
to implement the notification service in this project
```

```
@workspace follow C:/Users/<username>/.agents/skills/location_service_skill.md
and add location service with LocationCubit
```

---

## Claude AI (Web) Setup — claude.ai

### One-Time Setup

1. Go to [claude.ai](https://claude.ai)
2. In the left sidebar click **Projects**
3. Click **Create project** → name it **"Flutter Dev"**
4. Inside the project click **"Set project instructions"** (or the settings icon)
5. Paste the following into the instructions box:

```
You are a senior Flutter developer. Always follow these rules strictly:

ARCHITECTURE:
- Clean Architecture: domain (entities, usecases, repositories interface), data (models, datasources, repository impl), presentation (cubit/bloc, screens, widgets)
- Never put business logic in widgets or screens
- UseCase method name must be call()
- Repository: FeatureRepositoryImpl implements FeatureRepository
- DataSource: FeatureRemoteDataSourceImpl

API & NETWORK:
- Use ApiService with DioMethod enums
- Use executeApiRequest in repositories for Either<Failure, T>
- DataSource throws exceptions (BadRequestException etc) on failure
- Repositories map exceptions to Either and forward via Cubits

ERROR HANDLING:
- Use core failures from core/errors/failure.dart
- Show readable API error messages to user
- Use ResponseWidget for UI feedback

UI:
- Use AppTheme from core/theme/app_theme.dart
- Use context.textTheme and context.colors
- Reuse shared widgets: ButtonComponent, TextComponent, Loader, etc
- NEVER use CircularProgressIndicator — always use Skeletonizer for loading UI
- NEVER use showDialog directly — always use AppDialog static methods
- NEVER create new loaders — use Loader.show() / Loader.hide() / Loader.during()

STATE:
- Set loading status before async call
- Set success with payload on success
- Set error with message on failure

DI:
- Register as lazy singletons in service_locator.dart
- Register feature DI in feature injection file
- Register Cubit/Bloc as registerFactory

PAGINATION:
- Always use PaginationParams with page and limit
- Control pagination from Bloc only
- Append items on load more — never replace
- Stop when returnedItems < limit
- Never use API totalPages or totalItems

NAMING:
- Files: snake_case
- Classes: PascalCase
- Variables/methods: camelCase
- Constants: kConstantName
- Enums: PascalCase values
```

6. Click **Save**

### Every New Project

- All Flutter projects → use the same **Flutter Dev** project on claude.ai
- For a completely different domain project → create a new Project with separate instructions

### How to Use Skills with Claude AI

Start a conversation inside the Flutter Dev project and paste the skill content:

```
Use this notification service implementation pattern and set it up in my project:
[paste contents of notification_service_skill.md]
```

Or just describe what you need — the project instructions ensure it always follows your architecture.

---

## ChatGPT / Codex Setup

### One-Time Global Setup

1. Go to [chatgpt.com](https://chatgpt.com)
2. Click your **profile picture** (top-right corner)
3. Click **"Customize ChatGPT"**
4. In **"What would you like ChatGPT to know about you?"** paste:

```
I am a Flutter developer. I build production apps using:
- Clean Architecture (domain, data, presentation layers)
- flutter_bloc (Cubit and Bloc) for state management
- GetIt for dependency injection
- Dio + custom ApiService for networking
- fpdart Either<Failure, T> for error handling
- Repository pattern with UseCase layer
- skeletonizer package for loading UI (never spinners)
- Global AppDialog for all confirmation dialogs
- Global Loader overlay for action loading
My code follows strict clean architecture and naming conventions.
```

5. In **"How would you like ChatGPT to respond?"** paste:

```
Always generate Flutter code following these rules:

- Clean Architecture: domain / data / presentation layers
- UseCase method: call(). Repository: FeatureRepositoryImpl implements FeatureRepository
- Files: snake_case. Classes: PascalCase. Variables: camelCase
- State management: Bloc/Cubit only. Never use setState for business logic
- DI: GetIt lazy singletons. Cubit/Bloc as factory
- Pagination: PaginationParams(page, limit). Controlled from Bloc. Never use API totalPages
- On load more: items.addAll(newItems) — never replace list
- Loading UI: Skeletonizer(enabled: isLoading) — NEVER CircularProgressIndicator
- Action loading (submit/delete/update): Loader.show() / Loader.hide() / Loader.during()
- Dialogs: AppDialog.delete(), AppDialog.logout() etc — NEVER showDialog() directly
- Error: Either<Failure, T> in repository. Core failures in core/errors/failure.dart
- API: ApiService with DioMethod enums and executeApiRequest helper
Always generate complete files — entity, model, datasource, repository, usecase, cubit/bloc, screen, DI registration.
```

6. Click **Save**

### Every New Project

Global settings apply automatically. For project-specific context, paste this at the start of a new conversation:

```
I'm working on a Flutter project. Follow my Flutter clean architecture rules.
Project uses: [brief description of your project]
```

### How to Use Skills with Codex/ChatGPT

Paste the skill file content directly in the chat:

```
Implement a notification service in my Flutter project using this exact pattern:
[paste contents of notification_service_skill.md]

My project package name is: com.example.myapp
```

---

## How to Use Each Rule

Rules are hard constraints — the agent must never violate them.

| Rule File | What It Controls | When It Applies |
|---|---|---|
| `coding_rules.md` | Architecture layers, API pattern, DI, error handling | Every file generated |
| `folder_structure.md` | Where to place files inside `lib/` | Creating new features |
| `naming_conventions.md` | File names, class names, variable names, enums | Every line of code |
| `pagination_rules.md` | PaginationParams, Bloc-controlled pagination, load more append | Any paginated list |
| `loader_rules.md` | Global Loader.show/hide/during, no custom loaders | Any async action button |
| `app_dialog_rules.md` | AppDialog static methods, no showDialog | Any confirmation dialog |
| `skeletonizer_rules.md` | Skeletonizer for loading states, no CircularProgressIndicator | Any screen/list/card loading |

**How to activate a specific rule in chat:**

```
// GitHub Copilot
Follow the pagination rules: [paste pagination_rules.md content]
Build a paginated product list screen.

// Claude AI / ChatGPT
Use Skeletonizer for the loading state as per this rule: [paste skeletonizer_rules.md]
Build the orders list screen with proper skeleton loading.
```

---

## How to Use Each Skill

Skills are implementation guides — paste them when building a specific feature.

| Skill File | What It Builds | When to Use |
|---|---|---|
| `project_architecture.md` | Full clean arch overview, API flow, DI flow | When starting a new feature |
| `state_management.md` | Cubit/Bloc conventions, state patterns | Building any state layer |
| `firebase_remote_config_skill.md` | Remote Config singleton, model, feature flags | Adding remote config to project |
| `notification_service_skill.md` | FCM + local notifications, token sync, tap navigation | Setting up push notifications |
| `location_service_skill.md` | Geolocator service, LocationCubit, Google Maps geocoding | Adding location features |

**How to use a skill:**

```
Build a complete notification service for my Flutter project.
Use this exact implementation pattern:
[paste notification_service_skill.md content]

My package name: com.myapp.example
My StorageService has: getFcmToken() and saveFcmToken(String token)
```

---

## How to Use Workflows

Workflows are step-by-step processes — use them when building something from scratch.

| Workflow | What It Guides |
|---|---|
| `feature_workflow.md` | Build a complete feature: entity → datasource → repository → usecase → bloc → screen → DI → routes |
| `api_integration_workflow.md` | Integrate a new API endpoint into an existing feature |

**How to use:**

```
Follow this feature workflow and build a "Products" feature.
API endpoint: GET /products?page=1&limit=10
Response: { data: [{ id, name, price, imageUrl }] }

[paste feature_workflow.md content]
```

---

## How to Use Templates

Templates are copy-paste boilerplate — use them to generate files instantly.

| Template | Generates |
|---|---|
| `bloc_template.md` | Cubit class + State class with status enum and copyWith |
| `repository_template.md` | RepositoryImpl with executeApiRequest pattern |
| `screen_template.md` | Screen with BlocConsumer, loading, error, success states |

**How to use:**

```
Use this Bloc template and generate a ProductCubit for me:
[paste bloc_template.md content]

Feature name: Product
UseCase: GetProductsUseCase
Entity: ProductEntity
Fields: id (String), name (String), price (double), imageUrl (String)
```

---

## Updating Rules

When you update rules on one machine, sync to all machines:

```bash
# After editing any rule/skill file:
cd C:/Users/<username>/.agents
git add .
git commit -m "update: describe what changed"
git push

# On other machines:
cd C:/Users/<username>/.agents
git pull
```

---

## Quick Reference

| Tool | Global Setup | Per Project | Use Skills |
|---|---|---|---|
| **GitHub Copilot** | VS Code `settings.json` with file references | `.github/copilot-instructions.md` | `@workspace use skill from path` |
| **Claude AI (web)** | Project → Set Instructions | Same Flutter Dev project | Paste skill in chat |
| **ChatGPT / Codex** | Customize ChatGPT settings | Mention at start of chat | Paste skill in chat |
| **Claude Code (CLI)** | `~/.claude/CLAUDE.md` | Nothing needed | Automatic |
