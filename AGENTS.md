# AGENTS.md

These rules govern a coding agent's work in a project. Apply them alongside other applicable project instructions, respecting the priority of system, developer, and user instructions. Explicit user requests take precedence over the defaults in this file.

Favor correctness, minimal changes, and verifiable results. Use judgment for trivial tasks and avoid unnecessary process.

## 1. Think Before Coding

- State assumptions, constraints, and tradeoffs that materially affect implementation. Do not hide uncertainty.
- Ask focused clarification questions when different interpretations would materially change the result. Use and state reasonable defaults for minor details.
- Suggest simpler approaches when available, and challenge problematic requirements when warranted.
- Pause work that depends on missing essential information while continuing independent work.

## 2. Simplicity First

- Implement only what the task requires. Do not add speculative features.
- Avoid unnecessary abstractions, extensibility, or configuration for single-use code.
- Handle actual input constraints and failure modes; do not add defensive code for impossible scenarios.
- Prefer shorter implementations when they remain equally clear and satisfy all requirements.
- No interface with one implementation, no factory for one product, no configuration for a value that never changes.

## 3. Surgical Changes

- Every changed line should directly support the user's request.
- Do not improve adjacent code, comments, or formatting, or perform unrelated refactoring.
- Match the project's existing style.
- Remove unused imports, variables, and newly introduced code made obsolete by your changes.
- Report pre-existing dead code when relevant; do not delete it without authorization. Code of a feature that is no longer needed is the exception (Section 7).
- Preserve the user's existing changes. Do not overwrite or revert unrelated work.

## 4. Goal-Driven Execution

Before complex tasks, give a brief plan with verification criteria for key steps. For example:

1. Identify inputs and constraints → verify interfaces and callers.
2. Implement the minimal change → inspect the diff and behavior.
3. Validate the result → run relevant tests or reproducible checks.

- When fixing a bug, first create a reproducing test when practical.
- When adding validation, check valid inputs and relevant invalid inputs.
- For authorized refactoring, verify that behavior remains consistent before and after.
- Use proportionate verification for simple, reversible, low-risk changes. Do not add tests mechanically.
- Investigate and fix validation failures until the objective is met or an external blocker is established.
- Never claim that unexecuted tests passed. Report actual validation coverage and anything left unverified.

## 5. No Fallbacks; Fix the Root Cause

**Find the real cause of a bug and fix it there. Never paper over it with a fallback.**

- No fallback paths: no "if the field is missing, use the default", no "if this fails, try that instead", no silent retries, no default values that hide absent data, no `try/except` that swallows an error and carries on.
- Before fixing, reproduce the failure and trace it to the line where the wrong value or state first appears; fix that line, not the place where the symptom shows.
- A required value is required: read it unconditionally and let a missing one fail loudly. If a change makes existing data invalid, regenerate the data (section 7), do not add a reader-side shim.
- If the root cause cannot be fixed within the task, stop and report the cause and the options; do not ship a workaround.

Ask yourself: "Does this change remove the cause, or only hide the symptom?" If it only hides it, keep looking.

## 6. Reuse Existing Code

**Read the current implementation first. Do not reinvent what already exists.**

Before adding new code:

- Search the existing codebase for similar logic, helpers, utilities, patterns, and tests.
- Prefer reusing or extending existing functions/classes instead of creating parallel implementations.
- Match the current project’s abstractions and data flow unless there is a clear reason not to.
- If existing code already solves most of the problem, make the smallest necessary adaptation.
- Do not introduce a new dependency, framework, utility layer, or architecture when the current code can support the change.
- If the existing implementation is flawed but unrelated to the task, mention the issue rather than replacing it.

Ask yourself: "Am I solving this by fitting into the current codebase, or am I building a second system beside it?" If it is the second, stop and reuse what is already there.

## 7. Delete Unneeded Code; No Old-Data Compatibility

- Code for a feature that is no longer needed may be deleted outright, together with its tests, configs and docs. Do not keep it "just in case", behind a flag, or as an alias.
- Do not keep compatibility with old data formats: no reader-side shims, fallbacks for missing keys of old recordings, converters or version branches. Old recorded data is not supported. The only exception is when the user explicitly asks for compatibility with a specific format.

## 8. Naming Conventions

Names follow the Google Python Style Guide (naming 3.16, imports 2.2).

| Item | Rule |
|---|---|
| Underscore prefix | Avoid it: use public names, even for names used only inside their own module. Never on anything imported by another file, tests included (no `_merge_records()`) |
| Case | `snake_case` functions, modules and files; `CapWords` classes with all-caps acronyms (`IKConfig`, `VLMConfig`, `JSONClient`); `CAPS` constants |
| Verbs | `load_*` reads a file, `parse_*` turns a dict or text into an object, `get_*` looks up or computes by name, `is_*`/`has_*` return a bool, `build_*` constructs, `compute_*` calculates |
| Modules and packages | Helper modules are `<domain>_utils.py`; packages are plural nouns; exceptions end in `Error`; tests are `test_*.py` |
| Imports | Absolute imports of modules, not of classes or functions; no re-exports in `__init__.py` |
| Renames | Update every caller; never keep the old name as an alias |

## 9. Python Code

### 14.1 Pythonic and Elegant Implementation

- Use built-in language features and standard-library tools where appropriate.
- Prefer clear comprehensions, context managers, `pathlib`, `dataclasses`, and type hints when they simplify the code.
- Avoid Java/C++-style patterns that make Python code verbose or unnatural: no ABC or Protocol for a component with one implementation, no dependency-injection layers nobody swaps.
- Keep functions small, focused, and easy to read; do not sacrifice readability for clever one-liners.
- Write docstrings and comments in Google style: a one-line summary, then `Args:`, `Returns:`, `Raises:` sections for modules, classes, and public functions.

### 14.2 Reuse Existing Code

- Search the codebase for similar logic, helpers, utilities, patterns, and tests before adding code.
- Prefer reusing or extending existing functions and classes over parallel implementations; editing an existing function is fine when that is the smallest change.
- Do not introduce a new dependency, framework, utility layer, or architecture when the current code can support the change.
- When fixing or building, read the third-party references in `third_party/` first and adapt their code with the source cited; never modify `third_party/`.

### 14.3 Configuration

- Tunables live in YAML config groups under `<config>/<group>/` (`default.yaml`, variants beside it), loaded into dataclasses. Prefer an existing group.
- Never put tunables in module-level constants, global variables, function default arguments, or YAML files beside scripts.

### 14.4 Errors and Gates

- Do not use empty exception handlers or swallow exceptions; raise with useful context.
- Do not add protective checks for impossible cases. A new rejection gate needs the user's explicit approval.

## 10. Python Simplicity and Complexity

**Keep code as simple as the problem allows; add structure only when it pays for itself.** Extends sections 2, 5 and 9.

- Keep simple, cohesive logic in one function unless splitting improves reuse, composition, or readability. Do not split only to shorten functions.
- Validate unknown input once, at the boundary that receives it; pass typed, validated data inward. Do not re-check types, attributes, or invariants already guaranteed by validation or construction. Type hints do not validate at runtime; validate again only when crossing a new trust boundary or when a mutation may break an earlier guarantee.
- Do not extract a single-use helper early. Inline simple logic at the call site unless the helper is reused, isolates genuinely complex edge handling, or its name makes the call site clearer.
- Do not add abstractions, inheritance hierarchies, hooks, or configuration for hypothetical future needs. Define a Protocol or ABC only for a component with a real need to swap implementations, not for every implementation detail.
- Before adding complexity for a speculative or very unlikely race condition or security edge case, explain to the user the concrete failure, the evidence, how likely it is, and the complexity cost, and get their agreement. Never invent probability figures; do not widen scope for theoretical robustness.
- Avoid unnecessary `try/except`. Catch a specific exception only to recover, add useful context, translate an error at a boundary, or do required cleanup; otherwise let it propagate. Never swallow an exception silently; when translating, keep the chain (`raise ... from error`).
- Do not use `typing.Any` to bypass type checking. Prefer concrete types, generics, or a Protocol; use `object` for a truly unknown value until it is validated or narrowed.
- Public interfaces carry complete type hints, parameters and return values (with section 8, every name is public). Locals with obvious types may rely on inference.
- Choose a list comprehension, generator expression, or plain `for` loop by readability. Do not force `map`/`filter`, and do not squeeze complex logic into a comprehension; use an explicit loop for multi-step logic and side effects.
- Follow the project's existing package structure and import conventions; add no re-export layers to mimic other languages' module layouts (section 8).
- Keep dependency access and method calls readable: name an intermediate value when it explains a complex operation, and inject dependencies through the constructor.
- Inline a simple single-use expression when that reads better; keep an intermediate variable when it names business meaning, units, assumptions, or a non-obvious step.

Applies to new and changed code. Existing gaps (untyped public functions, `typing.Any`, broad `except`) are cleaned up as a separate task.

## 11. Subagent Collaboration

This file explicitly authorizes and requests delegation of concrete implementation work when subagent capabilities are available and the task benefits from decomposition.

- The primary agent owns requirements analysis, task decomposition, interface coordination, integration, and final verification.
- Delegate independent code exploration, implementation, testing, and bulk editing where useful.
- Use the subagent tools actually available in the current environment. Do not assume Claude's `Agent` tool or Opus or Sonnet models exist.
- Complete simple tasks directly, and work directly when subagents are unavailable.
- Assign clear file ownership, inputs, outputs, and acceptance criteria. Avoid concurrent edits to the same files.
- Include applicable rules in delegation briefs, including simplification, review, verification, and Python constraints.
- Do not substitute user-visible new chats for internal subagents.
- A subagent's completion report does not replace the primary agent's verification of critical results.

## 12. Simplify, Review, and Validate Changes

After each batch of code changes, follow this order:

1. Implement the change.
2. Perform a behavior-preserving simplification pass over the diff.
3. Review the diff and fix actionable findings.
4. Run relevant tests or other validation.
5. Commit only when requested or already authorized by the user.

- If an applicable `simplify` or code-review skill is available, read its instructions and use it.
- Do not assume that `/simplify`, `code-review`, or a skill with either name exists.
- When those capabilities are unavailable, perform equivalent checks directly: complexity, duplication, correctness, interface compatibility, error handling, and test coverage.
- Include fixes from simplification and review in the final checks and validation.
- Apply the same workflow to subagent changes.
- Before an authorized push, confirm that the changes being pushed have been reviewed. This workflow does not itself authorize commits or pushes.

## 13. Worktrees and Completed Branches

- Create every git worktree under `.claude/worktrees/` in the project root, never beside the repository.
- Land work on the integration branch (for example `dev`) by rebase and fast-forward; no merge commits.
- Once a task is merged and its checks pass, delete its local branch and worktree. Do not keep completed branches as backups.
- Never remove a worktree that holds uncommitted or unmerged work; keep it until the user authorizes discarding it. Never delete the integration or release branch (for example `dev`, `main`).
- Delete remote branches only within the user's authorization for that remote.

## 14. Branch Names

Use a short branch name of at most three words, separated by hyphens, for example `box-home-retreat`. No slashes and no type prefixes such as `feat/` or `fix/`. Worktrees take the branch name as their directory name under `.claude/worktrees/`.
## 15. Use Chinese by Default

- Answer in Chinese unless the user explicitly asks for another language.
- Code, shell commands, identifiers, error logs, and proper nouns (papers, libraries, APIs, models, projects) stay in English; keep an English term and explain it in Chinese when that is more precise.

## 16. Response Style

- Lead with the answer, followed by only the details needed to act.
- Prefer short tables for parallel results and short paragraphs for simple explanations.
- For experiments or runs, report relevant actual inputs, such as code version, configuration, task, device, random seeds, and data, together with output files, metrics, or decisions.
- Omit inapplicable fields. Never invent unknown values.
- List and explain output locations separately according to Section 17.
- Report implementation changes, validation results, and unresolved limitations without unrelated process narration.

## 17. File and Directory Paths

- Use absolute paths when referencing actual files or directories. Put each path on its own line and explain its purpose.
- Use clickable absolute-path links when supported.
- Identify the host for remote paths. Do not assume remote paths are locally accessible.
- Do not invent absolute paths that have not been confirmed.
- Project-relative paths in this file are location conventions; resolve them against the confirmed project root during execution.

## 18. Document Style

- Keep documents short. Prefer tables for information that benefits from comparison.
- Use plain, precise language. Briefly explain unavoidable internal names.
- Explain what files and directories contain or are used for; do not list unexplained paths.
- State the current status, decisions, and results. Avoid commit-by-commit narratives or histories of attempted approaches.
- Apply these requirements to documents produced by subagents as well.

## 19. IEEE TPAMI Paper Format

| Item | Requirement |
|---|---|
| LaTeX class | `\documentclass[10pt,journal,compsoc]{IEEEtran}` |
| Figure and table width | Use `\columnwidth` for one column and `\textwidth` with `figure*` for spanning figures. Check the template's actual dimensions. |
| Captions | Figure captions below figures; table captions above tables. Explain what the reader should learn from the figure or table. |
| Tables | Use booktabs top, middle, and bottom rules, no vertical lines, and bold for the best result. |
| Front matter | Abstract followed by Index Terms. |
| References | IEEE numbered style using IEEEtran.bst, ordered by first citation. |
| Notation and terminology | Define symbols and abbreviations on first use. Use one consistent name for each concept. |
| Prose | Short sentences and active voice. Use present tense for method behavior, include units, and use consistent precision within each table. |

Explicit submission templates or user requirements take precedence over these project defaults.

