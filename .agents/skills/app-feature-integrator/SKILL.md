---
name: app-feature-integrator
description: Investigate a feature in an application with REA, specify its observable behavior, and implement an independently tested equivalent module in the current project.
metadata:
  version: "1"
---

# App Feature Integrator

Use this workflow when the user wants to understand a feature in another
application and build equivalent functionality in their own project.

Do not use this workflow for ordinary edits to a source repository when the
feature is already clear from the available source. For analysis-only requests,
use the existing `reverse-engineer-anything` skill without forcing implementation.

## Architecture rule

Keep three responsibilities separate:

- **REA engine (MCP/CLI):** inspects the authorized target and returns
  observations, evidence, and limitations.
- **This Skill:** coordinates investigation, specification, implementation,
  and verification.
- **Destination module:** contains the new feature's production code and
  integrates with the destination app.

REA is a development-time analysis tool by default. Do not embed REA, its CLI,
or an analysis engine into the shipped app unless the user explicitly requests
that product behavior. Do not assume `rea-agents` exposes a stable importable
library API; use its documented CLI/MCP interface unless a supported public API
is verified.

## Operating rules

1. Work only on applications and data the user owns or is authorized to inspect.
2. Prefer documented APIs and SDKs when they provide the required behavior.
3. Reimplement behavior independently. Do not copy proprietary source code, credentials, private assets, or secrets into the destination project. Do not bypass licensing, authentication, DRM, or access controls.
4. Inspect the destination repository before choosing paths, frameworks, or
   module names. Follow existing conventions; `modules/<feature-slug>/` and
   `specs/<feature-slug>.md` are examples, not mandatory directories.
5. Do not claim that analysis recovers the complete original source, exact
   algorithm, or hidden server implementation unless evidence establishes it.
6. Keep observations, inferences, and unknowns distinct. Preserve source paths,
   hashes, Evidence IDs, tool/provider details, and coverage limitations where
   available.
7. Use only tools exposed by the active REA MCP server and their advertised
   schemas. Do not invent tool names or inputs. Check the current CLI help/docs
   when using CLI commands.

## Workflow

### 1. Confirm both sides of the work

Resolve from the current conversation/workspace:

- **Source target:** application, package, website, binary, or runtime that contains the behavior being studied.
- **Feature:** the exact user-visible capability to reproduce, including a representative input/output when available.
- **Destination:** the current project that will receive the equivalent implementation.
- **Constraints:** expected platform, language, API/provider choice, compatibility, performance, and whether UI wiring is in scope.

Use paths and context already supplied. If a missing detail is essential and
cannot be inferred from the active workspace, ask one focused question rather
than guessing the target.

### 2. Discover available analysis capabilities

- Check the current session's REA tool list and schemas before calling tools.
- Prefer an existing `rea` CLI. If unavailable, use
  `npx -y rea-agents@latest` only when installing/running the package is within
  the user's scope.
- Route by target type using the existing `reverse-engineer-anything` skill
  and REA guides.
- If REA is unavailable, continue with supported local inspection and state the
  limitation. Do not block all progress on optional native engines when static
  analysis is sufficient.

### 3. Investigate narrowly and preserve evidence

Start with the smallest investigation that can answer the feature question.
Prefer static analysis before runtime interaction. Use runtime capture only
when the user has authorized the target and the action is required to
understand behavior.

Capture, as applicable:

- inputs, outputs, options, defaults, and visible states;
- module boundaries, call/data flow, IPC/API boundaries, and relevant
  dependencies;
- error handling, retries, cancellation, and edge cases;
- source evidence, target digest, tool/provider identity, and analyzed coverage.

Do not fetch or contact remote endpoints found in a target merely because they
appear in its files or traffic capture. Do not broaden a runtime action beyond
the stated target and task.

### 4. Write a feature specification before coding

Create or update a concise specification in the destination repo, following
its documentation conventions. Include:

- goal and acceptance criteria;
- observed behavior and representative examples;
- inferred behavior, confidence, and unresolved questions;
- proposed input/output contract and error model;
- dependencies, platform constraints, security/privacy considerations;
- evidence references and coverage limitations;
- tests required to demonstrate compatibility.

If source evidence is incomplete, design an explicit fallback or mark the behavior as unknown. Do not silently convert guesses into requirements.

### 5. Design the destination module

Inspect the destination architecture first. Define a small public interface
and keep the feature isolated from unrelated application concerns.

The module should have, as appropriate:

- typed inputs and outputs;
- explicit configuration rather than hidden global state;
- consistent error/result semantics;
- cancellation, timeouts, and progress reporting for long operations;
- bounded resource use and clear ownership/cleanup;
- logging that excludes secrets and sensitive payloads;
- dependency injection for external APIs, files, and model providers.

Use an official API/SDK where suitable. Otherwise implement equivalent
observable behavior from the specification. Do not import the source
application's private modules or REA internals into the production module.

### 6. Implement and test

- Add the smallest implementation that meets the acceptance criteria.
- Add unit tests for core logic, integration tests for boundaries, and regression tests for representative cases.
- Cover invalid inputs, empty data, errors/timeouts, cancellation, and platform-specific behavior where relevant.
- Compare results against the specification and available reference
  observations. Explain any intentional or unavoidable differences.
- Run the relevant lint, typecheck, build, and test commands discovered in the repository.
- Do not claim tests passed unless they were actually run and their result was observed.

### 7. Integrate only what was requested

If UI integration was requested, connect the new module using existing
UI/state patterns. Otherwise leave UI changes out and document the module's
public entry point.

Do not add REA to the user's app runtime merely to implement a feature learned
through REA. If the user explicitly wants an in-app analysis product, treat
that as a separate architecture decision with isolated process supervision,
resource limits, user consent, and clear engine dependencies.

### 8. Report the result

Finish with:

- files created or changed;
- what behavior is implemented;
- evidence used and important unknowns;
- tests/commands run and their actual outcomes;
- setup or configuration required;
- deviations from the source behavior.

Distinguish a behavior-compatible independent implementation from a literal
source-code reconstruction.
