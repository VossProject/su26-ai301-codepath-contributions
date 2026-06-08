# Contribution 1: invalid length specifier for Integer declaration

**Contribution Number:** 1  
**Student:** Mikey Voss  
**Issue:** https://github.com/lfortran/lfortran/issues/3855  
**Status:** Phase I Complete

---

## Why I Chose This Issue

I want to build a compiler from scratch eventually, so contributing to one that already works feels like a great way to learn how a real compiler is put together before I attempt to write my own. Fortran is also used a lot for simulations across many STEM fields, so the compiler behind that work is something I'd like to help improve.

LFortran is a stable, active codebase, and this bug is small and well defined: the compiler accepts an invalid length specifier on an integer (`INTEGER :: i*2`) and silently ignores it instead of reporting an error. That matters because a compiler that quietly swallows invalid code hides mistakes from the programmer, which is the opposite of its job. I picked something small on purpose. I don't know Fortran yet, but this issue is more about compiler error handling than Fortran itself, and keeping the first contribution low risk means I can aim for more than one and take on something more ambitious on my next pass.

---

## Understanding the Issue

### Problem Description

LFortran accepts a character-length specifier (`*N`) on a non-character type and ignores it instead of rejecting it. In `INTEGER, parameter :: i*2 = 1`, the `*2` is a length specifier. Length specifiers only mean something for `CHARACTER`, where they set how many characters a string holds, so attaching one to an `INTEGER` is invalid. LFortran drops the `*2` and compiles the line as if it were `i = 1`, with no error or warning.

### Expected Behavior

The compiler should raise a semantic error saying the length specifier is not valid for an integer declaration. Per the issue, the same error should fire whether or not `parameter` is present, so plain `INTEGER :: i*2` should also be rejected.

### Current Behavior

LFortran ignores the `*2`, compiles successfully, and prints `1`. No diagnostic is emitted.

### Affected Components

Semantic analysis of declarations, where the parsed AST is lowered to ASR (LFortran's typed semantic representation). The exact file and function are to be confirmed during reproduction; I have not traced the code yet.

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]

1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

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
