# Contribution 1: invalid length specifier for Integer declaration

**Contribution Number:** 1  
**Student:** Mikey Voss  
**Issue:** https://github.com/lfortran/lfortran/issues/3855  
**Status:** Phase II Complete

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

A compile-time error goes in `tests/errors/continue_compilation_1.f90` (reference test), not an integration test (`AGENTS.md`). Planned for Phase III:

- [ ] `integer, parameter :: i*2 = 1` raises the error.
- [ ] `integer :: i*2` raises the error.
- [ ] `character :: x*3` still compiles (no regression to the character path).

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**

- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [LFortran issue #3855](https://github.com/lfortran/lfortran/issues/3855)
- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
