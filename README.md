# CodePath AI301 — Open Source Contribution Log

**Contributor:** A.T.M. Hossain ([@ATMmonitor667](https://github.com/ATMmonitor667))
**Program:** CodePath AI301 — AI Open Source Capstone (Section 1b)
**Status:** Phase I Complete

| | |
|---|---|
| **Project** | [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw) |
| **Issue** | [#9527 — Display error message for invalid hexadecimal color codes](https://github.com/excalidraw/excalidraw/issues/9527) |
| **My fork** | [ATMmonitor667/excalidraw](https://github.com/ATMmonitor667/excalidraw) |
| **Pull request** | [#11499](https://github.com/excalidraw/excalidraw/pull/11499) |
| **Language / stack** | TypeScript, React, SCSS |

---

## Phase I — Issue Selection

### Issue

[excalidraw/excalidraw#9527 — Improve Feedback: Display Error Message for Invalid Hexadecimal Color Codes](https://github.com/excalidraw/excalidraw/issues/9527)

### Problem Summary

Excalidraw's color picker has a hex input field that silently ignores invalid values. If you type something that can't be parsed as a color (too many digits, invalid characters, or plain text like `blue`), the input is dropped with no visual or textual feedback, so the user is left confused about why the color didn't change. This matters because silent failure is a usability and accessibility gap — good input validation should tell the user what went wrong. "Fixed" means the field surfaces a clear, accessible error message when the entered value is not a valid color, and clears it once a valid value is entered.

### Why I Chose This Issue

I chose issue #9527 because it's a well-scoped front-end validation task that aligns with my TypeScript/React experience and my goal of strengthening my UX and accessibility skills. It's labeled `good first issue` and `UX/UI`, the maintainers are clearly active, and the affected area (the color picker's `ColorInput` component) is small and self-contained — a realistic first contribution rather than a sprawling refactor.

Specifically:

1. **Skill match.** I've built form/input validation before in personal projects, so I understand the core pattern (validate input then reflect state in the UI), and the rest of the React/TS specifics are things I can ramp on quickly.
2. **Bounded scope.** The change is contained to one component plus a styling file and a localization string — there's a clear definition of "done" in the issue's Expected Behavior section.
3. **Active maintenance.** Excalidraw is a large, actively-maintained project (100k+ stars) with a strong `CONTRIBUTING.md` and devcontainer, so setup and review are well-supported.
4. **Learning goal.** I want to learn how a mature codebase handles input normalization (`normalizeInputColor`) and how it wires user-facing strings through its i18n layer.

From reading the issue, I understand the root cause: `ColorInput` calls `normalizeInputColor(value)`, and when that returns `null` (invalid), the code simply doesn't call `onChange` — so nothing visible happens. My contribution adds an `isValid` state, marks the field `aria-invalid`, and renders an inline error message, improving both the user experience and accessibility.

I have already reproduced the behavior locally, implemented a fix, and opened [PR #11499](https://github.com/excalidraw/excalidraw/pull/11499) for it.

---

## Phase II — Understanding the Issue

### Understanding the issue
_To be completed in Phase II._

### Reproduction process
_To be completed in Phase II._

### Solution approach
_To be completed in Phase II._

---

## Phase III — Build

### Testing strategy
_To be completed in Phase III._

### Implementation notes
_To be completed in Phase III._

---

## Phase IV — Pull Request

### PR link
[excalidraw/excalidraw#11499 — feat(editor): show inline error for invalid color input](https://github.com/excalidraw/excalidraw/pull/11499)

### Summary
Added inline validation feedback to the color picker's hex input: when a non-empty value can't be parsed as a color, the input is marked `aria-invalid`, gets a red error border, and shows a "Not a valid color" message below it. The message clears on blur or once a valid color is entered. Changes span `ColorInput.tsx`, `ColorPicker.scss`, and the `en.json` locale.

### Maintainer feedback log
_To be updated as review progresses._
