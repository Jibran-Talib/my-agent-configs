# Personal AI Agent Config

My personal rules, skills, workflows, and templates for Flutter development.

## Structure

```
.agents/
├── flutter_dev.agent.md        # Main agent definition
├── rules/                      # Coding rules and standards
│   ├── coding_rules.md
│   ├── folder_structure.md
│   ├── loader_rules.md
│   ├── naming_conventions.md
│   └── pagination_rules.md
├── skills/                     # Architecture and patterns
│   ├── project_architecture.md
│   └── state_management.md
├── workflows/                  # Step-by-step workflows
│   ├── feature_workflow.md
│   └── api_integration_workflow.md
└── templates/                  # Code templates
    ├── bloc_template.md
    ├── repository_template.md
    └── screen_template.md
```

---

## New Machine Setup

Kisi bhi naye laptop pe yeh repo clone karo:

```bash
git clone https://github.com/Jibran-Talib/my-agent-configs.git C:/Users/<username>/.agents
```

---

## Claude Code (CLI) Setup

### One-time Global Setup

1. `~/.claude/CLAUDE.md` file mein saari rules paste karo
2. Ya directly copy karo:

```bash
cat C:/Users/<username>/.agents/rules/*.md > C:/Users/<username>/.claude/CLAUDE.md
```

3. Ab har project mein Claude Code automatically yeh rules follow karega

### Har Naye Project Mein

Kuch karne ki zaroorat nahi — `~/.claude/CLAUDE.md` globally apply hota hai.

---

## Claude AI (Web) Setup — claude.ai

### One-time Setup

1. [claude.ai](https://claude.ai) kholo
2. Left sidebar mein **Projects** click karo
3. **New Project** banao — name: `Flutter Dev`
4. Project ke andar **Project Instructions** click karo
5. `rules/` folder ke saare files ka content paste karo
6. Save karo

### Har Naye Project Mein

- Naya Flutter project ho toh same **Flutter Dev** project use karo
- Agar alag domain ho toh naya Project banao aur wahan instructions paste karo

---

## GitHub Copilot Setup

### One-time Global Setup (VS Code)

1. VS Code kholo
2. `settings.json` mein yeh add karo:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "text": "Always follow clean architecture with domain, data, presentation layers. Use Bloc for state management. Use repository pattern. Follow snake_case for files, PascalCase for classes. Use PaginationParams for all paginated APIs. Control pagination from Bloc only. Use global Loader.show() and Loader.hide() — never use CircularProgressIndicator directly."
    }
  ]
}
```

### Har Naye Project Mein

Project root mein yeh file banao:

```
.github/copilot-instructions.md
```

Content:

```markdown
You are a senior Flutter developer.

Follow these rules strictly:

- Clean architecture: domain, data, presentation layers
- State management: Bloc/Cubit only
- Repository pattern with Either<Failure, T>
- UseCase method name: call()
- snake_case for files, PascalCase for classes
- Pagination controlled from Bloc only using PaginationParams
- Global Loader.show() / Loader.hide() — no CircularProgressIndicator
- Register all DI in service_locator.dart
- Use AppTheme, context.textTheme, context.colors for UI
```

---

## ChatGPT / Codex Setup

### One-time Global Setup

1. [chatgpt.com](https://chatgpt.com) kholo
2. Top-right corner mein apna **Profile** click karo
3. **Customize ChatGPT** click karo
4. **"What would you like ChatGPT to know about you?"** mein likho:

```
I am a Flutter developer. I use clean architecture with domain, data, and presentation layers.
I use Bloc for state management and repository pattern with Either<Failure, T>.
My projects follow strict naming conventions and folder structure.
```

5. **"How would you like ChatGPT to respond?"** mein paste karo:

```
Always follow clean architecture: domain, data, presentation layers.
Use Bloc/Cubit for state management only.
Use repository pattern. UseCase method must be call().
File names in snake_case. Classes in PascalCase.
Pagination must use PaginationParams and be controlled from Bloc only.
Never use API totalPages or totalItems.
Always use Loader.show() / Loader.hide() — never CircularProgressIndicator.
Register all dependencies in service_locator.dart.
Use AppTheme for UI consistency.
```

6. Save karo

### Har Naye Project Mein

- Global settings already apply hongi
- Agar project-specific context dena ho toh conversation start mein likho:
  `"Follow my Flutter clean architecture rules for this project."`

---

## Rules Update Karna

Jab bhi rules update karo:

```bash
cd C:/Users/<username>/.agents
git add .
git commit -m "update rules"
git push
```

Dusre machine pe sync karo:

```bash
cd C:/Users/<username>/.agents
git pull
```

---

## Quick Reference

| Tool | Global Setup | Per Project |
|---|---|---|
| **Claude Code (CLI)** | `~/.claude/CLAUDE.md` | Kuch nahi |
| **Claude AI (web)** | Project > Instructions | Same project use karo |
| **GitHub Copilot** | VS Code settings.json | `.github/copilot-instructions.md` |
| **ChatGPT / Codex** | Customize ChatGPT | Conversation mein mention karo |
