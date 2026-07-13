# Contribution 2: issue selection in progress

**Contribution Number:** 2  
**Student:** Mikey Voss  
**Issue:** not selected yet  
**Status:** Phase I In Progress

---

## Why I Chose This Issue

No issue picked yet, so this section is the selection log until there is one.

Started from the July 10 refresh of the course candidate sheet: about 6,500 rows, 4,900 of them unclaimed, across 502 repos. Shortlisted around my languages (Java, Python) and interests (compiler work after the lfortran PR, AI tooling, running my own server).

Vetted 11 shortlisted issues across cpython, zulip, micrometer, prettier, llvm-project, and jellyfin. For each one I read the full comment thread, checked assignees, and pulled linked pull requests from GitHub's timeline API.

Ten of the eleven failed the same check: an open PR already solving the issue, most under active maintainer review. The eleventh was an unbounded cleanup meta-issue that three merged PRs had already picked over. The sheet's Claimed column only tracks our cohort, so it can't see in-flight PRs from the rest of the internet.

| Repo | Issue | What it is | Why I passed |
|------|-------|------------|--------------|
| python/cpython | [#73065](https://github.com/python/cpython/issues/73065) | smtplib should add a Date header per RFC 5322 | open PR, approved, waiting on a core dev |
| python/cpython | [#130821](https://github.com/python/cpython/issues/130821) | inconsistent TypeError messages from conversion dunders | main fix merged, leftovers owned by two stale open PRs |
| zulip/zulip | [#38436](https://github.com/zulip/zulip/issues/38436) | nested bullet lists break with tab indentation | two open PRs, one in active review |
| micrometer-metrics/micrometer | [#5066](https://github.com/micrometer-metrics/micrometer/issues/5066) | warn when an instrumented cache is not recording stats | Caffeine case merged, JCache and Hazelcast PRs parked for 1.18 |
| llvm/llvm-project | [#205306](https://github.com/llvm/llvm-project/issues/205306) | clang assertion crash on a wide string literal to `__builtin_nanf` | open PR in review, claim contested by two others |
| llvm/llvm-project | [#189583](https://github.com/llvm/llvm-project/issues/189583) | `computeKnownFPClass` missing `ISD::SQRT` handling | open PR in review |
| llvm/llvm-project | [#189764](https://github.com/llvm/llvm-project/issues/189764) | nullptr deref when a profile file is missing | open two-line PR already covers it |
| prettier/prettier | [#19528](https://github.com/prettier/prettier/issues/19528) | markdown printer inserts a stray `>` before `\>` | three open PRs racing, maintainer engaged on one |
| jellyfin/jellyfin | [#16258](https://github.com/jellyfin/jellyfin/issues/16258) | POST /Auth/Keys returns 204 with no key in the body | open PR iterating after maintainer review |
| jellyfin/jellyfin | [#8971](https://github.com/jellyfin/jellyfin/issues/8971) | add a playback speed property to the state APIs | open PR with three approvals |
| jellyfin/jellyfin | [#5355](https://github.com/jellyfin/jellyfin/issues/5355) | remove unnecessary `ToList` calls | four-year meta-issue, already picked over, no bounded slice left |

LLVM got ruled out on policy rather than availability. Its [AI tool policy](https://llvm.org/docs/AIToolPolicy.html) forbids using AI tools to fix good-first-issues (searching and learning with LLMs is fine), and the bot restates that on every issue with the label. This course's workflow is AI-assisted, so that label is off the table.

Next: re-run selection against the low-traffic repos on the sheet (emfcamp/Website, music21, tm_devices, cartography), filtering for zero linked PRs before reading anything else. Claim, fork, and comment the same day this time.

---

# Contribution 1: invalid length specifier for Integer declaration

**Contribution Number:** 1  
**Student:** Mikey Voss  
**Issue:** https://github.com/lfortran/lfortran/issues/3855  
**Status:** Phase IV Complete (PR merged)

---

## Why I Chose This Issue

I want to build a compiler from scratch eventually, so contributing to one that already works is a good way to learn how a real one fits together first. Fortran also runs a lot of STEM simulation work, so the compiler behind it is worth improving.

This bug is small and well defined: LFortran accepts an invalid length specifier on an integer (`INTEGER :: i*2`) and silently ignores it instead of erroring. A compiler that quietly swallows invalid code hides mistakes from the programmer, the opposite of its job. I picked something small on purpose. I don't know Fortran yet, but this is more about error handling than Fortran, and keeping the first one low risk lets me aim for several contributions.

---

## Understanding the Issue

### Problem Description

In `INTEGER, parameter :: i*2 = 1`, the `*2` is a length specifier, which is only valid on `CHARACTER`. LFortran drops it on the integer and compiles the line as `i = 1`, with no error or warning.

### Expected Behavior

A semantic error. Per the issue, it should fire with or without `parameter`, so plain `INTEGER :: i*2` is invalid too.

### Current Behavior

It compiles and prints `1`. No diagnostic.

### Affected Components

AST-to-ASR semantic analysis of declarations, in `src/lfortran/semantics/ast_common_visitor.h` (line 7351). Details in Solution Approach.

---

## Reproduction Process

### Environment Setup

My machine had none of the build tools (no CMake, Ninja, re2c, or LLVM). I followed the documented "Build From Git" path in `doc/src/installation.md` rather than installing each dependency by hand, since the build scripts assume a conda-style environment.

1. Installed micromamba with Homebrew (`AGENTS.md` recommends it over full Conda).
2. Built the pinned `lf` environment from `environment_linux.yml` (LLVM 11, re2c, bison, CMake, Ninja, zlib, zstd).
3. Ran `./build0.sh` (re2c/bison/Python code generation), then `./build1.sh` (CMake + Ninja). Binary at `src/bin/lfortran`.

Only snag: I expected LLVM 11 to be missing on Apple Silicon, but conda-forge has an arm64 build, so it resolved first try.

Platform: macOS arm64. LFortran `0.63.0-706-g9fe913970`.

Working branch: https://github.com/VossProject/lfortran/tree/fix-issue-3855

### Steps to Reproduce

Assumes a built `src/bin/lfortran` (see Environment Setup).

1. Save this program as `repro.f90`:

   ```fortran
   program repro
       integer, parameter :: i*2 = 1
       print *, i
   end program repro
   ```

2. Compile and run it: `./src/bin/lfortran repro.f90`
3. **Observed:** the program prints `1` and exits 0. No error or warning is emitted.
4. **Expected:** a semantic error reporting that a length specifier is not valid on an integer.
5. Repeat with `parameter` removed, to confirm the bug is not specific to constants:

   ```fortran
   program repro
       integer :: i*2
       i = 1
       print *, i
   end program repro
   ```

   This also prints `1` with no error.

I ran each variant twice and got `1` every time.

### Reproduction Evidence

- **Working branch:** https://github.com/VossProject/lfortran/tree/fix-issue-3855
- **Commit:** the snippets above are the minimal example; the regression test lands in Phase III.
- **Logs:**

  ```
  $ ./src/bin/lfortran repro.f90
  1
  $ echo $?
  0
  ```

- **ASR:** `--show-asr` types `i` as plain `(Integer 4)` with no length in the tree. The `*2` never reached the semantics.

---

## Solution Approach

### Analysis

The parser stores the `*2` in the entity's `var_sym.m_length` field. The per-entity declaration loop in `src/lfortran/semantics/ast_common_visitor.h` reads it at line 7351:

```cpp
if (is_char_type && s.m_length) {
    // apply the length to the character type
}
```

There is no `else`. On a non-character type with a length set, it falls through and the `*2` is dropped, no diagnostic.

### Proposed Solution

Add the missing `else`: when a length is set on a non-character type, raise a semantic error with the file's standard idiom (`diag.add(...)` then `throw SemanticAbort();`). The message names the type via `ASRUtils::type_to_str_fortran`, e.g. `length specifier is not allowed for type 'integer'`. The loop runs per entity, so it fires with or without `parameter`.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Raise a semantic error instead of silently dropping a length specifier on a non-character type, with or without `parameter` (root cause in Analysis above).

**Match:** The `if (is_char_type && s.m_length)` block at 7351 (from PR #3791, old-style `character :: x*3`) is the prior art. Copy the `diag.add(...)` + `throw SemanticAbort();` idiom used across this file.

**Plan:**

1. Add an `else if (s.m_length)` branch after line 7351.
2. Build the message with `ASRUtils::type_to_str_fortran(type)`, `diag.add(...)` at `Error`/`Semantic`, then `throw SemanticAbort();`.
3. Add both variants to `tests/errors/continue_compilation_1.f90` (appended) and update references.

**Implement:** Phase III. Branch: https://github.com/VossProject/lfortran/tree/fix-issue-3855

**Review:** Per `AGENTS.md`: lowercase message, explicit type, no ASR node names; one bug/one PR, no unrelated changes; `fixes #3855` against `upstream/main`.

**Evaluate:** Test fails on `main`, passes on the branch. Run `./run_tests.py` plus integration tests for regressions.

---

## Testing Strategy

A compile-time error goes in `tests/errors/continue_compilation_1.f90` (reference test), not an integration test (`AGENTS.md`). Done in Phase III:

- [x] `integer, parameter :: i*2 = 1` raises the error.
- [x] `integer :: i*2` raises the error.
- [x] `character :: x*3` still compiles (no regression to the character path).

Both failing cases live in a new subroutine appended at the end of the file, with distinct names (`i` and `j`). The file compiles in continue-compilation mode, which recovers after each error and keeps collecting, so reusing one name would have stacked a spurious "already declared" diagnostic on top of the real one.

The fix fails on `main` and passes on the branch. On `main` both cases print `1` and exit 0. On the branch each raises `semantic error: length specifier is not allowed for type 'integer'`, and the regenerated reference matches. I checked for wider impact two ways: a grep of the whole test corpus shows every other `name*N` declaration is on `CHARACTER`, the path I left untouched, and the `character2` reference test still passes.

The full reference suite has two local failures unrelated to this change: a macOS linker warning about a dylib built for a newer SDK, and a `.F90` C-preprocessor macro that does not expand the way it does on the project's Linux CI. Both fail the same way on clean `main`.

---

## Implementation Notes

### Phase III Progress

Started by syncing the fork. It was 159 commits behind `upstream/main`, and upstream had rewritten both the file I needed to edit and the test file I planned to extend, so my Phase II line numbers were stale. Synced first, rebuilt, then re-located the fix site in `src/lfortran/semantics/ast_common_visitor.h`.

The fix is small. The per-entity declaration loop already had an `if (is_char_type && s.m_length)` block that applies a length to `CHARACTER`, but no `else`, so a length on any other type fell through and was dropped. I added an `else if (s.m_length)` branch that raises a semantic error and throws `SemanticAbort`. It runs once per entity, so it fires with or without `parameter`.

### Code Changes

- **Files modified:**
  - `src/lfortran/semantics/ast_common_visitor.h` (the fix)
  - `tests/errors/continue_compilation_1.f90` and its two regenerated reference files (the test)
- **Key commits:**
  - [`5e2e366`](https://github.com/VossProject/lfortran/commit/5e2e366bb) fix: reject length specifier on non-character type
  - [`7c29acf`](https://github.com/VossProject/lfortran/commit/7c29acf5c) test: add length specifier on non-character error cases
- **Approach decisions:**
  - Pointed the error at the entity location (`s.loc`), not the whole statement, so a multi-entity declaration like `integer :: a, i*2, b` underlines `i*2` instead of the entire line.
  - Used the message `length specifier is not allowed for type 'integer'` without the kind. `AGENTS.md` says to show explicit kinds for type errors, but the kind is irrelevant here: the error is about the length syntax, not whether the type is `integer(4)` or `integer(8)`.

### Challenges Faced

The fork was 159 commits behind upstream, and upstream had rewritten both files I needed, so my Phase II line numbers were stale. Had to sync and re-find the fix site before writing anything.

Two snags after that. My test run reported green when it had actually failed, because I passed `-j16` and the top-level runner doesn't take that flag (it's for the integration runner). The log showed the failure, the exit code didn't. The other one: the full suite failed on a macOS linker warning and a `.F90` preprocessor quirk, neither related to my change, both failing on clean `main` too. Spent a while sure I'd broken something before I actually read the diffs.

**Branch:** https://github.com/VossProject/lfortran/tree/fix-issue-3855

---

## Pull Request

**PR Link:** https://github.com/lfortran/lfortran/pull/12058

**PR Description:** Made LFortran raise a semantic error when a length specifier (`i*2`) is placed on a non-character type, instead of silently dropping it and compiling the line as `i`. One-line fix in the declaration semantics, plus a regression test covering both the plain and `parameter` forms.

**Maintainer Feedback:**

- @opixdown reviewed and left three notes: drop the `! {Error}` inline comments (the stderr already shows the error), collapse my two test cases into one (`integer :: i*2` and the `parameter` version both hit the same `else if (s.m_length)` branch, so it was the same check twice), and reword the message to `length specifier is only valid for character type`. All three were fair. Addressed them in one follow-up commit and he gave an LGTM.
- After that `main` moved and `continue_compilation_1.f90` (the shared error-test file everyone appends to) conflicted. Merged `upstream/main` in, re-resolved, kept CI green.
- Then it just sat, green and reviewed. Learned the hard way that a reviewer's LGTM isn't a merge, and that the "Contributor" badge next to someone's name says nothing about whether they can merge. A polite nudge on the thread moved it along.

**Status:** Merged 2026-07-04 by @opixdown (merge commit `d9405da8`).

---

## Resources Used

- [LFortran issue #3855](https://github.com/lfortran/lfortran/issues/3855)
