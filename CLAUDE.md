# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

* State your assumptions explicitly. If uncertain, ask.
* If multiple interpretations exist, present them - don't pick silently.
* If a simpler approach exists, say so. Push back when warranted.
* If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

* No features beyond what was asked.
* No abstractions for single-use code.
* No "flexibility" or "configurability" that wasn't requested.
* No error handling for impossible scenarios.
* If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

* Don't "improve" adjacent code, comments, or formatting.
* Don't refactor things that aren't broken.
* Match existing style, even if you'd do it differently.
* If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

* Remove imports/variables/functions that YOUR changes made unused.
* Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

* "Add validation" → "Write tests for invalid inputs, then make them pass"
* "Fix the bug" → "Write a test that reproduces it, then make it pass"
* "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```text
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Pythonic and Elegant Implementation

**Write idiomatic, readable Python. Prefer clarity over cleverness.**

When writing Python:

* Use built-in language features and standard-library tools where appropriate.
* Prefer clear comprehensions, context managers, `pathlib`, `dataclasses`, and type hints when they simplify the code.
* Avoid Java/C++-style patterns that make Python code verbose or unnatural.
* Keep functions small, focused, and easy to read.
* Do not sacrifice readability for overly clever one-liners.
* Choose names that make the code self-explanatory.
* Write docstrings and comments in Google style: a one-line summary, then `Args:`, `Returns:`, `Raises:` sections for modules, classes, and public functions.

Ask yourself: "Would an experienced Python engineer consider this idiomatic and maintainable?" If not, simplify and make it more Pythonic.

## 6. Reuse Existing Code

**Read the current implementation first. Do not reinvent what already exists.**

Before adding new code:

* Search the existing codebase for similar logic, helpers, utilities, patterns, and tests.
* Prefer reusing or extending existing functions/classes instead of creating parallel implementations.
* Match the current project’s abstractions and data flow unless there is a clear reason not to.
* If existing code already solves most of the problem, make the smallest necessary adaptation.
* Do not introduce a new dependency, framework, utility layer, or architecture when the current code can support the change.
* If the existing implementation is flawed but unrelated to the task, mention the issue rather than replacing it.

Ask yourself: "Am I solving this by fitting into the current codebase, or am I building a second system beside it?" If it is the second, stop and reuse what is already there.

## 7. Use Chinese by Default

**Answer in Chinese by default unless the user explicitly requests another language.**

When responding:

* Use Chinese by default for explanations, code analysis, implementation plans, debugging steps, and summaries.
* If the user asks for translation, polishing, or content generation in English or another language, follow the requested language.
* Code, shell commands, variable names, function names, error logs, and technical terms may remain in English.
* Do not force-translate proper nouns such as paper titles, library names, API names, model names, or project names.
* When mixed Chinese-English terminology improves accuracy, keep the English term and explain it in Chinese.

Ask yourself: "Did the user explicitly request an English response?" If not, respond in Chinese.

## 8. Naming Conventions

**Google Python Style Guide names; LIBERO-style vocabulary.**

| Item | Rule |
|---|---|
| Underscore prefix | Only for names used inside their own module. Anything imported by another file (including tests) gets a public name, never `_merge_xxx()` |
| Case | `snake_case` functions, modules and files; `CapWords` classes with all-caps acronyms (`IKConfig`, `GPUBatch`); `CAPS` constants |
| Verbs | `load_*` reads a file, `parse_*` turns a dict or text into an object, `get_*` looks up or computes by name, `is_*`/`has_*` return a bool, `build_*` constructs |
| Modules and packages | Helper modules are `<domain>_utils.py`; packages are plural nouns; exceptions end in `Error`; tests are `test_*.py` |
| Imports | Absolute imports of modules, not of classes or functions; no re-exports in `__init__.py` |
| Renames | Update every caller; never keep the old name as an alias |
| Config | Tunables live in YAML config groups (`<config>/<group>/default.yaml`, variants beside it), never in module-level constants, global variables, function defaults or YAML files scattered beside scripts |

## 9. Delegate Implementation Work

Your primary responsibilities are analysis, orchestration, and verification. Delegate concrete tasks to subagents through the Agent tool whenever possible.

Limit your own work to clarifying requirements, breaking down plans, assigning tasks, and reviewing results. Delegate implementation work—including extensive code reading, coding, test execution, and bulk edits—to subagents.

## 10. Simplify and Review Every Change

Order per change: edits → `simplify` → `code-review` → tests → commit.

* After any code change, run the `simplify` skill once over the diff and apply what it finds as long as behaviour is preserved. Every change gets one pass, not only complex ones.
* Then run the `code-review` skill once over the same diff and fix what it finds.
* This applies to every commit, including subagents' commits (put it in their briefs) and commits from other sessions, which are reviewed before pushing.

## 11. Delete Unneeded Code; No Old-Data Compatibility

* Code for a feature that is no longer needed may be deleted outright, together with its tests, configs and docs. Do not keep it "just in case", behind a flag, or as an alias.
* Do not keep compatibility with old data formats: no reader-side shims, fallbacks for missing keys of old recordings, converters or version branches, unless the user explicitly asks for a specific format.

## 12. Give Full Paths, One per Line

Whenever a reply names a file or directory (videos, figures, logs, reports, configs), give its full absolute path, never a relative path or a bare filename. Put each path on its own line, for example in a code block, never inline in a sentence or a table cell. For a path on a remote host, give the full remote path and name the host.

## 13. Write Documents Short, in Tables, Without History

This applies to every document (wiki pages, reports, write-ups), including those a subagent writes.

* Prefer tables wherever the content allows.
* Explain every path: say what it is or holds, never a bare path.
* Use plain words. Avoid internal names the reader does not know; if one is unavoidable, explain it in a few words.
* Keep documents short.
* Do not write history. State the current state, decisions and results, not how they evolved.

## 14. Reply in Short Tables with Clear Inputs and Outputs

Replies follow the same style as documents (section 13):

* Prefer short tables over paragraphs.
* For every task, change, run or result, state the input (code version, config, data, seeds), the output (files, numbers, decision) and where the output is.
* Lead with the answer, then only the details needed to act on it.

## 15. Draw Every Figure in the TPAMI Style

Every figure, for a paper or for analysis, uses the project's TPAMI matplotlib style file (`tpami.mplstyle`), applied with `plt.style.use(...)` before the figure is created. This style wins over any generic chart guidance.

| Item | Rule |
|---|---|
| Size | Final print size, never rescaled afterwards: width 3.5 in for one column, 7.16 in for two columns |
| Font | Serif (Times New Roman, falling back to Nimbus Roman or STIX), math in STIX; 9 pt for labels and titles, 8 pt for ticks and legends |
| Lines | Axes and ticks 0.6 pt, data lines 1.0 pt, markers 3.5 pt; no grid |
| Colours | Only five colour-blind-safe colours: blue 0072B2, orange E69F00, green 009E73, vermillion D55E00, black; sequential data uses the Blues colormap |
| Greyscale | Separate the series by marker or line style as well as by colour, so the figure still reads in print |
| Legend | Square box with a light grey edge, fully opaque |
| Labels on data | Text over plotted data gets a white box: `bbox={"facecolor": "white", "edgecolor": "none", "pad": 1}` |
| Output | Both PDF (vector) and PNG (600 dpi), saved with `bbox_inches="tight"` |
| Location | `figures/<intent>/<name>/`, where `<intent>` is `paper` or `analysis` |
| Tests and benchmarks | Every test or benchmark run also produces the figures of whatever it measured |

## 16. Write Papers in the IEEE TPAMI Format

| Item | Rule |
|---|---|
| LaTeX class | `\documentclass[10pt,journal,compsoc]{IEEEtran}`: two columns |
| Figure and table width | One column 3.5 in (`\columnwidth`), two columns 7.16 in (`\textwidth`, `figure*`); figures are drawn at that size (section 15) |
| Captions | Figure captions below the figure ("Fig. 1."), table captions above the table; a caption says what to read off the figure, not only what it shows |
| Tables | `booktabs` rules (`\toprule`, `\midrule`, `\bottomrule`), no vertical lines; best result in bold |
| Front matter | Abstract followed by "Index Terms" keywords |
| References | IEEE numbered style (`IEEEtran.bst`), cited in order of appearance |
| Notation and terms | Define every symbol and abbreviation at first use, and keep one name per concept throughout (text, figures and tables) |
| Prose | Short sentences, active voice, present tense for what the method does; numbers with units and the same precision across a table |

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
