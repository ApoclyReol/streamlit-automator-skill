---
name: streamlit-dev-workflow
description: Use when the user wants a project-agnostic workflow for building, reorganizing, or maintaining a local Python Streamlit app. This skill defines a reusable Streamlit engineering pattern for startup bootstrapping, env-based configuration, layered architecture, workflow-oriented UI design, local persistence boundaries, and optional macOS launcher packaging without assuming any specific product domain or schema.
---

# Streamlit Development Workflow

This skill defines a **general local-development pattern** for Python Streamlit projects.

Use it when the user wants to:

- start a new Streamlit project with a clean structure
- reorganize an older Streamlit repo into clearer layers
- standardize `run.sh`, `.env`, and local dependency setup
- separate data, business logic, UI, and infrastructure concerns
- design Streamlit navigation around workflows instead of raw objects
- package the project as a simple local macOS launcher app

Do **not** use it when the user actually wants:

- a domain-specific product spec
- cloud deployment as the main objective
- a true native desktop app
- a cross-platform installer or release pipeline
- frontend architecture advice unrelated to Streamlit

## Core stance

Treat the skill as a **project-independent engineering template**.

It should define:

- decision order
- recommended directory roles
- layer boundaries
- startup responsibilities
- configuration boundaries
- packaging boundaries

It should **not** define:

- business modules
- app-specific page names
- product-specific database tables
- domain formulas
- a requirement to rename every repo into one exact structure

If a repo already exists, map the current structure onto the template before proposing changes.

## Recommended project shape

This is a recommended structure, not a mandatory literal tree:

- `app/`: application entrypoint and page assembly
- `views/` or `pages/`: page containers that orchestrate UI interactions
- `components/`: reusable UI fragments
- `services/`: business logic and workflow orchestration
- `repositories/` or `data_access/`: persistence and query boundaries
- `core/`: config, paths, logging, runtime bootstrap, shared infrastructure
- `data/`: local databases, caches, exports, generated runtime files
- `scripts/`: operational scripts such as import/export/init/maintenance helpers
- `docs/`: agent and maintainer documentation
- `run.sh`: standard local entrypoint

When applying this template:

- prefer mapping current folders into these roles over forced renaming
- only rename when the current structure is actively confusing
- preserve working code paths while clarifying boundaries

## Layer responsibilities

### Entry layer

Responsible for:

- app bootstrapping
- configuration initialization
- top-level route or page dispatch
- one-time startup hooks

Should not contain:

- business formulas
- raw SQL
- page-specific data manipulation

### Page layer

Responsible for:

- layout
- form controls
- user interactions
- state coordination
- calling services

Should not contain:

- direct SQL
- schema migration logic
- core domain formulas duplicated across pages

### Component layer

Responsible for:

- reusable UI fragments
- rendering helpers
- repeated editor or chart widgets

Should not contain:

- persistence logic
- cross-page business orchestration

### Service layer

Responsible for:

- business rules
- calculations
- import/export orchestration
- workflow-level data shaping
- coordination across repositories

Should not contain:

- page layout code
- environment-loading code scattered per feature

### Data layer

Responsible for:

- connection management
- queries
- writes
- transactions
- persistence abstractions

Should not contain:

- UI state logic
- page-triggered ad hoc SQL embedded elsewhere

### Core layer

Responsible for:

- environment and config loading
- paths
- shared constants
- logging
- common infrastructure

Should not contain:

- product-specific page flow
- business state persistence mixed with infrastructure defaults

## Startup contract

`run.sh` is the standard project entrypoint.

Its responsibilities are:

1. resolve the project root
2. verify Python availability
3. verify `uv` availability
4. check whether `.venv` exists
5. create `.venv` when missing
6. install or sync dependencies
7. launch the Streamlit entrypoint

Default installation stance:

- prefer `uv`
- do not require the user to activate the virtual environment manually
- keep startup idempotent
- make `run.sh` suitable for development, daily local usage, and launcher wrapping

If a project lacks `run.sh`, create one before discussing packaging.

## Configuration contract

Use the project root `.env` as the local configuration source.

Rules:

- system environment variables take precedence
- `.env.example` is the public template
- code should read configuration through a centralized config layer
- `.env` is only for runtime configuration and secrets

Do not store in `.env`:

- business facts
- primary user data
- versioned structured master data

Typical `.env` categories:

- API keys
- model names and API endpoints
- DB paths
- debug flags
- optional runtime switches

## UI design contract

Do not make the skill assume concrete business pages.

Instead, use these design principles:

- organize top-level navigation around workflows
- each workflow page should solve one primary user task
- separate data entry/import from results/analysis
- separate configuration/maintenance from day-to-day operation
- use global context selectors as support, not as the entire navigation model

Recommended workflow-oriented navigation template:

- data input
- processing and review
- results and analysis
- configuration and maintenance

These are templates, not required page names.

## Persistence contract

The skill should define persistence boundaries without assuming one domain schema.

Default recommendation:

- prefer SQLite for local single-user Streamlit apps

Required principles:

- there must be a clear DB initialization path
- schema changes require an explicit migration strategy
- import/export/backup flows must stay aligned with schema changes
- data access should be centralized
- page code must not build arbitrary SQL inline

If a project uses another local persistence strategy, preserve the same layer boundaries.

## Packaging contract

Treat macOS packaging as an optional local convenience layer.

Default packaging mode:

- use `osacompile`
- create a Terminal-opening launcher app
- wrap `./run.sh`
- do not reproduce dependency-install logic inside AppleScript

Important framing:

- this is a local launcher, not a true desktop distribution format
- app naming should come from the current project context
- packaging should happen only after `run.sh` is stable

Preferred template:

```bash
osacompile -o /Applications/<APP_NAME>.app \
  -e 'on run' \
  -e 'tell application "Terminal"' \
  -e 'activate' \
  -e 'do script "cd <PROJECT_DIR> && ./run.sh"' \
  -e 'end tell' \
  -e 'end run'
```

## Decision order

When using this skill, follow this sequence:

1. identify the current project shape
2. map folders into entry/page/component/service/data/core roles
3. verify or create a standard `run.sh`
4. centralize configuration into `.env` and config loading
5. separate UI, business logic, and persistence boundaries
6. redesign navigation around workflows when needed
7. package into a local launcher app only after the above is stable

## Validation checklist

The resulting project should satisfy these checks:

- deleting `.venv` does not prevent `./run.sh` from preparing the environment
- config can be loaded from `.env` with environment override support
- business logic is not implemented in page files
- SQL or persistence calls are not spread across UI code
- the app can still be launched through one clear Streamlit entrypoint
- launcher packaging, if used, only wraps `./run.sh`

## Recommended close-out pattern

Summaries produced with this skill should stay generic and implementation-focused:

- what project shape was identified
- how the current repo maps onto the template
- whether startup is standardized
- whether config loading is centralized
- whether UI and persistence boundaries are clean
- whether launcher packaging is ready or intentionally deferred
