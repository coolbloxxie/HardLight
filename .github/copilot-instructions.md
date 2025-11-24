Purpose
-------
This file helps AI coding agents become immediately productive in the HardLight repository.

Quick Architecture Summary
-------------------------
- **Engine / Framework**: `RobustToolbox` provides the underlying game engine and shared runtime helpers.
- **Content split**: `Content.Server`, `Content.Client`, and `Content.Shared` hold game logic for server, client, and cross-cutting types respectively.
- **Resources & Assets**: `Resources/` contains textures and data; `ProgramShared` mounts them at runtime depending on the `contentStart` flag.
- **ECS + Systems**: The codebase uses an Entity-Component-System style. Systems typically live in `EntitySystems` folders and are named `*System` (example: `Content.Server/Chemistry/EntitySystems/InjectorSystem.cs`).

Key Workflows (build, run, test)
--------------------------------
- SDK: The repo targets .NET SDK declared in `global.json` (SDK `9.x`). Use the `dotnet` toolchain.
- Build (local): `dotnet build` or use the ready VS Code/Tasks labels `build` / `build-release`.
- Run server: `dotnet run --project Content.Server` (or `runserver.bat` / `runserver.sh`).
- Run client: `dotnet run --project Content.Client` (or `runclient.bat` / `runclient.sh`).
- Unit tests: use the provided test task or run manually, e.g.
  - `dotnet test --no-build --configuration DebugOpt Content.Tests/Content.Tests.csproj`
  - Integration tests: `dotnet test --no-build --configuration DebugOpt Content.IntegrationTests/Content.IntegrationTests.csproj`

Project-specific Patterns & Conventions
-------------------------------------
- Namespaces and layout: Code is organized by feature under `Content.*`. Shared cross-project types live in `Content.Shared`.
- Systems: Use dependency injection via `[Dependency] private readonly ...` fields, subscribe to events with `SubscribeLocalEvent<TComp, TEvent>(...)`, and raise events with `RaiseLocalEvent`.
- Entities / Components: Components live in `Components` folders. Entity identifiers use `EntityUid` or the `Entity<TComponent>` generic shorthand in systems.
- Localization: Use `Loc.GetString("key", ("arg", value))` for user-facing strings.
- UI / Popups / Chat: Use `Popup.PopupEntity(...)` for in-game messages and `IChatManager` for chat channels.
- Long actions: Use `DoAfter.TryStartDoAfter(...)` for delayed user interactions (injection, dragging, etc.).
- Solutions / Chemistry: Chemical handling uses `SolutionContainers` helpers (split/draw/inject) and `ReactiveSystem` for reactions — see `InjectorSystem.cs` for a real example.

Common Integration Points / External Dependencies
------------------------------------------------
- Engine/Tooling: `RobustToolbox` is imported and contains important helpers (see `RobustToolbox/Robust.Shared/ProgramShared.cs`).
- Packages: `Directory.Packages.props` centralizes package versions (OpenTK, Veldrid, EF core overrides). Do not add package versions ad-hoc—update `Directory.Packages.props`.
- Data mounts: `ProgramShared` logic changes paths when launching with `contentStart`; tests and engine runs may require using the `contentStart` behavior.

Useful Example References (read before changing patterns)
-------------------------------------------------------
- Systems & ECS: `Content.Server/Chemistry/EntitySystems/InjectorSystem.cs` — shows dependency injection, `DoAfter`, `Popup`, `SolutionContainers`, `AdminLogger`.
- Entry points: `Content.Server/Program.cs` and `RobustToolbox/Robust.Server/Program.cs` — how `ContentStart` and content mounting are decided.
- Packaging & SDK: `global.json` and `Directory.Packages.props`.
- Run helpers: `runserver.bat`, `runclient.bat`, `runserver.sh`, `runclient.sh`.
- Tests: `Content.Tests/`, `Content.IntegrationTests/` and workspace tasks (`test`, `integration-test`).

When Editing Code — Practical Rules for an AI
---------------------------------------------
- Follow existing folder and naming conventions; prefer adding new systems/components under `Content.*` not `RobustToolbox` unless engine changes are required.
- Avoid changing centralized versioning files (`Directory.Packages.props`) without explicit reason.
- Preserve localization keys; add new keys to the appropriate `.resx`/localization files rather than hardcoding strings.
- Use `DoAfter` for actions that should be interruptible; copying patterns from `InjectorSystem` is a good template.
- When changing gameplay logic, run the relevant unit/integration tests and, if applicable, `dotnet run` the server and client to exercise interactions.

If anything is unclear or you'd like this tailored (e.g., more examples, test commands, or CI details), tell me which section to expand.
