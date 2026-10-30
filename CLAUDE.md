# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it. Code of a feature that is no longer needed is the exception (section 7).

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

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

## 8. Project Naming Conventions

Names follow the Google Python Style Guide (naming 3.16, imports 2.2).

| Item | Rule |
|---|---|
| Underscore prefix | Avoid it: use public names, even for names used only inside their own module. Never on anything imported by another file, tests included (no `_merge_records()`) |
| Case | `snake_case` functions, modules and files; `CapWords` classes with all-caps acronyms (`IKConfig`, `VLMConfig`, `JSONClient`); `CAPS` constants |
| Verbs | `load_*` reads a file, `parse_*` turns a dict or text into an object, `get_*` looks up or computes by name, `is_*`/`has_*` return a bool, `build_*` constructs, `compute_*` calculates |
| Modules and packages | Helper modules are `<domain>_utils.py`; packages are plural nouns; exceptions end in `Error`; tests are `test_*.py` |
| Imports | Absolute imports of modules, not of classes or functions; no re-exports in `__init__.py` |
| Renames | Update every caller; never keep the old name as an alias |

## 9. Pythonic and Elegant Implementation

**Write idiomatic, readable Python. Prefer clarity over cleverness.**

When writing Python:

- Use built-in language features and standard-library tools where appropriate.
- Prefer clear comprehensions, context managers, `pathlib`, `dataclasses`, and type hints when they simplify the code.
- Avoid Java/C++-style patterns that make Python code verbose or unnatural.
- Keep functions small, focused, and easy to read.
- Do not sacrifice readability for overly clever one-liners.
- Choose names that make the code self-explanatory.
- Write docstrings and comments in Google style: a one-line summary, then `Args:`, `Returns:`, `Raises:` sections for modules, classes, and public functions.

Ask yourself: "Would an experienced Python engineer consider this idiomatic and maintainable?" If not, simplify and make it more Pythonic.

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

## 11. Delegate Implementation Work

Your primary responsibilities are analysis, orchestration, and verification. Delegate concrete tasks to subagents, such as Opus or Sonnet, whenever possible.

Limit your own work to clarifying requirements, breaking down plans, assigning tasks, and reviewing results. Delegate implementation work—including extensive code reading, coding, test execution, and bulk edits—to subagents through the Agent tool.

## 12. Run /simplify After Every Code Change

After finishing any code change and before running the tests and committing, invoke the `simplify` skill once over the change (the diff or the changed files) and apply what it finds as long as behaviour is preserved. This is not optional and not reserved for changes that look complex: every change gets one simplification pass.

Then invoke the `code-review` skill once over the same change and fix what it finds, before running the tests and committing. Order per change: edits → `simplify` → `code-review` → tests → commit. This applies to every commit, including subagents' commits (put it in their briefs) and commits from other sessions, which are reviewed before pushing.

## 13. Worktrees and Completed Branches

- Create every git worktree under `.claude/worktrees/` in the project root, never beside the repository.
- Land work on the integration branch (for example `dev`) by rebase and fast-forward; no merge commits.
- Once a task is merged and its checks pass, delete its local branch and worktree. Do not keep completed branches as backups.
- Never remove a worktree that holds uncommitted or unmerged work; keep it until the user authorizes discarding it. Never delete the integration or release branch (for example `dev`, `main`).
- Delete remote branches only within the user's authorization for that remote.

## 14. Branch Names

Use a short branch name of at most three words, separated by hyphens, for example `box-home-retreat`. No slashes and no type prefixes such as `feat/` or `fix/`. Worktrees take the branch name as their directory name under `.claude/worktrees/`.
## 15. Use Chinese by Default

**Answer in Chinese by default unless the user explicitly requests another language.**

When responding:

- Use Chinese by default for explanations, code analysis, implementation plans, debugging steps, and summaries.
- If the user asks for translation, polishing, or content generation in English or another language, follow the requested language.
- Code, shell commands, variable names, function names, error logs, and technical terms may remain in English.
- Do not force-translate proper nouns such as paper titles, library names, API names, model names, or project names.
- When mixed Chinese-English terminology improves accuracy, keep the English term and explain it in Chinese.

Ask yourself: "Did the user explicitly request an English response?" If not, respond in Chinese.

## 16. Reply in Short Tables with Clear Inputs and Outputs

Replies to the user follow the same style as documents (section 18):
- Prefer short tables over paragraphs.
- For every task, change, run or result, state the key factors explicitly: the input (code version, config, task, robot, seeds, data), the output (files, numbers, decision), and where the output is (paths, explained per section 18).
- Keep it brief: lead with the answer, then only the details needed to act on it.

## 17. Give Full Paths, One per Line

Whenever a reply names a file or directory (videos, figures, logs, reports, configs), give its full absolute path, never a relative path or a bare filename. Put each path on its own line, for example in a code block, never inline in a sentence or a table cell. For paths on a remote host such as the 4090, give the full remote path and say which host it is on.

## 18. Write Documents Short, in Tables, Without History

This applies to every document (wiki pages, reports, write-ups), including those a subagent writes.
- Prefer tables wherever the content allows.
- Explain every path: say what it is or holds (for example "the GPU server's output directory of this run"), never a bare path.
- Use plain words. Avoid internal names the reader does not know; if one is unavoidable, explain it in a few words.
- Keep documents short.
- Do not write history. State the current state, decisions and results, not how they evolved (no commit-by-commit narration, no "first tried X, then reverted").

## 19. Draw Every Figure in the TPAMI Style

Every figure, for a paper or for analysis, uses the project's TPAMI matplotlib style file (`tpami.mplstyle`). This style wins over any generic chart guidance.

| Item | Rule |
|---|---|
| Style file | `plt.style.use(<path to tpami.mplstyle>)`, before creating the figure |
| Size | Final print size, never rescaled afterwards: width 3.5 in for one column, 7.16 in for two columns |
| Font | Serif (Times New Roman, falling back to Nimbus Roman or STIX), math in STIX; 9 pt for labels and titles, 8 pt for ticks and legends |
| Lines | Axes and ticks 0.6 pt, data lines 1.0 pt, markers 3.5 pt; no grid |
| Colours | Only the style's five, colour-blind safe: blue 0072B2, orange E69F00, green 009E73, vermillion D55E00, black; sequential data uses the Blues colormap |
| Greyscale | Separate the series by marker or line style as well as by colour, so the figure still reads in print |
| Legend | Square box with a light grey edge, fully opaque |
| Labels on data | Text over plotted data gets a white box: `bbox={"facecolor": "white", "edgecolor": "none", "pad": 1}` |
| Output | Both PDF (vector, for the paper) and PNG (600 dpi), saved with `bbox_inches="tight"` |
| Location | `figures/<intent>/<name>/`, where `<intent>` is `paper` (paper figures) or `analysis` (diagnosis and checking) |
| Tests and benchmarks | Every test or benchmark run also produces the figures of whatever it measured |

## 20. Write the Paper in the IEEE TPAMI Format

| Item | Rule |
|---|---|
| LaTeX class | `\documentclass[10pt,journal,compsoc]{IEEEtran}`: TPAMI is an IEEE Computer Society journal, two columns |
| Figure and table width | One column 3.5 in (`\columnwidth`), two columns 7.16 in (`\textwidth`, `figure*`); figures are drawn at that size (section 19) |
| Captions | Figure captions below the figure ("Fig. 1."), table captions above the table; a caption says what to read off the figure, not only what it shows |
| Tables | `booktabs` rules (`\toprule`, `\midrule`, `\bottomrule`), no vertical lines; best result in bold |
| Front matter | Abstract followed by "Index Terms" keywords |
| References | IEEE numbered style (`IEEEtran.bst`), cited in order of appearance |
| Notation and terms | Define every symbol and abbreviation at first use, and keep one name per concept throughout (the same name in text, figures and tables) |
| Prose | Short sentences, active voice, present tense for what the method does; numbers with units and the same precision across a table |

