# CodePath AI301 — Open Source Contribution Log

**Contributor:** A.T.M. Hossain ([@ATMmonitor667](https://github.com/ATMmonitor667))
**Program:** CodePath AI301 — AI Open Source Capstone (Section 1b)
**Status:** Phase III Complete

| | |
|---|---|
| **Project** | [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw) |
| **Issue** | [#9527 — Display error message for invalid hexadecimal color codes](https://github.com/excalidraw/excalidraw/issues/9527) |
| **My fork** | [ATMmonitor667/excalidraw](https://github.com/ATMmonitor667/excalidraw) |
| **Working branch** | [fix/9527-invalid-hex-feedback](https://github.com/ATMmonitor667/excalidraw/tree/fix/9527-invalid-hex-feedback) |
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
3. **Active maintenance.** Excalidraw is a large, actively-maintained project (100k+ stars) with a strong `CONTRIBUTING.md`, so setup and review are well-supported.
4. **Learning goal.** I want to learn how a mature codebase handles input normalization (`normalizeInputColor`) and how it wires user-facing strings through its i18n layer.

From reading the issue, I understand the root cause: `ColorInput` calls `normalizeInputColor(value)`, and when that returns `null` (invalid), the code simply doesn't call `onChange` — so nothing visible happens. My contribution adds an `isValid` state, marks the field `aria-invalid`, and renders an inline error message, improving both the user experience and accessibility.

---

## Phase II — Reproduce & Plan

### Reproduction Process

#### Environment Setup

Excalidraw does not ship a dev container, so I followed the standard README / `CONTRIBUTING.md` setup (the "typical case"):

1. Cloned my fork and checked out my working branch.
2. Confirmed I had **Node ≥ 18** (the version range pinned in `package.json`) and **Yarn 1.22** (the repo's declared `packageManager`).
3. Ran `yarn install` to install dependencies.
4. Started the app with `yarn start`, which runs the Vite dev server for `excalidraw-app` at `http://localhost:3000`.

> _Personalize this: add any specific errors you hit during setup and how you resolved them (e.g. a Node version mismatch fixed with `nvm use`, or a port-in-use conflict)._

**Working branch:** https://github.com/ATMmonitor667/excalidraw/tree/fix/9527-invalid-hex-feedback

#### Steps to Reproduce

1. Run `yarn start` and open `http://localhost:3000`.
2. Select a drawing tool (e.g. the rectangle or draw tool) so the left-hand styles panel appears.
3. Click the **stroke** (or background) color swatch to open the color picker.
4. In the hex input field, enter an invalid value — for example `zzzzzz`, `12345`, or `blue`.
5. **Expected:** a clear, visible error message indicating the value is not a valid color.
6. **Actual:** the input is silently ignored — the selected color does not change and no error or feedback is shown.

Reproduced consistently across several invalid inputs (wrong length, non-hex characters, and plain words), confirming it's not a fluke.

### Solution Approach

#### Implementation Plan (UMPIRE)

**Understand.** The hex color input accepts text but gives no feedback when the value can't be parsed as a color. The user can't tell whether their input was rejected or why — a usability and accessibility gap.

**Match.** The same `ColorInput` component already uses React state (`innerValue`), conditional class names via `clsx`, and pulls user-facing strings through the i18n helper `t(...)`. The color-parsing logic already exists in `normalizeInputColor` (`packages/common/src/colors.ts`), which returns `null` for invalid input — I can reuse that return value as my validity signal rather than writing new parsing or regex.

**Plan.**
1. Add an `isValid` boolean state to `ColorInput.tsx`.
2. In `changeColor`, set `isValid` based on whether `normalizeInputColor` returned a value, treating an empty field as not-an-error.
3. Mark the `<input>` `aria-invalid={!isValid}`, and reset validity on blur and when the `color` prop changes.
4. Render an inline error message below the input when the value is invalid.
5. Add styling for the error message / invalid border in `ColorPicker.scss`.
6. Add a `colorPicker.invalidColor` string to `packages/excalidraw/locales/en.json`.

**Files I expect to touch:**
- `packages/excalidraw/components/ColorPicker/ColorInput.tsx`
- `packages/excalidraw/components/ColorPicker/ColorPicker.scss`
- `packages/excalidraw/locales/en.json`

**Implement.** (Phase III) Work happens on the branch above: [fix/9527-invalid-hex-feedback](https://github.com/ATMmonitor667/excalidraw/tree/fix/9527-invalid-hex-feedback).

**Review.** Self-review against the project's `CONTRIBUTING.md`; run `yarn fix` (Prettier + ESLint) and follow Excalidraw's commit-message / PR conventions before opening the PR.

**Evaluate.** Re-run the reproduction steps and confirm an error now appears for invalid input and clears once a valid color is entered; run `yarn test:typecheck` (tsc) and `yarn test` (Vitest) to confirm no regressions; run `yarn test:other` (Prettier check) to confirm formatting.

---

## Phase III — Build

### Testing strategy

This is a small front-end UX/accessibility change with no new business logic, so I verified it through a mix of manual testing and the project's existing automated checks:

- **Manual before/after.** Re-ran the Phase II reproduction. Before: invalid hex (`zzzzzz`, `12345`, `blue`) was silently ignored. After: the input gets a red border and an inline "Not a valid color" message, which clears when a valid color is entered or the field loses focus.
- **Edge cases (manual).** Empty field shows no error (treated as incomplete, not invalid); valid 3/6/8-digit hex and `transparent` are accepted; changing the selected element resets validity via the `color` prop effect.
- **Type checking.** `yarn test:typecheck` (tsc) passes — the new state and `aria-invalid` are correctly typed.
- **Existing test suite.** `yarn test` (Vitest) passes with no regressions in the color picker or surrounding components.
- **Formatting / lint.** `yarn test:other` (Prettier `--list-different`) and `yarn fix` confirm the diff matches the project's style.
- **Accessibility.** Confirmed the input exposes `aria-invalid="true"` when invalid and the message uses `role="alert"`, so assistive tech announces it.

### Implementation notes

The change lives entirely in the color picker and touches three files.

**`components/ColorPicker/ColorInput.tsx`**
- Added an `isValid` boolean state (default `true`).
- In `changeColor`, derived validity from the existing `normalizeInputColor` result: an empty field is treated as not-an-error, any other unparseable value sets `isValid` to `false`.
- Added `aria-invalid={!isValid}` to the `<input>`.
- Reset `isValid` to `true` on blur and whenever the `color` prop changes, so the error doesn't linger after a valid selection.
- Rendered an inline error (`role="alert"`) below the input when the value is invalid.

**`components/ColorPicker/ColorPicker.scss`**
- Styled the error message and an invalid-state border using the existing `--color-danger` theme variable (defined for both light and dark themes), so it matches Excalidraw's design system.

**`locales/en.json`**
- Added the user-facing string under `colorPicker.invalidColor`, wired through the existing `t(...)` i18n helper instead of hard-coded text.

**Key decisions**
- **Reused `normalizeInputColor`** (`packages/common/src/colors.ts`) as the single source of truth for validity rather than writing a separate regex, so the feedback can never disagree with what the picker actually accepts.
- **Kept the diff minimal** and consistent with the component's existing patterns (React state, `clsx`, `t()`), making it easy for maintainers to review.

**Commit / PR hygiene.** Work was done on the `fix/9527-invalid-hex-feedback` branch and submitted as a single, focused commit in [PR #11499](https://github.com/excalidraw/excalidraw/pull/11499).

---

## Phase IV — Pull Request

### PR link
[excalidraw/excalidraw#11499 — feat(editor): show inline error for invalid color input](https://github.com/excalidraw/excalidraw/pull/11499)

### Summary
Added inline validation feedback to the color picker's hex input: when a non-empty value can't be parsed as a color, the input is marked `aria-invalid`, gets a red error border, and shows a "Not a valid color" message below it. The message clears on blur or once a valid color is entered. Changes span `ColorInput.tsx`, `ColorPicker.scss`, and the `en.json` locale.

### Maintainer feedback log
_To be updated as review progresses._
# CodePath AI301 — Open Source Contribution Log

**Contributor:** A.T.M. Hossain ([@ATMmonitor667](https://github.com/ATMmonitor667))
**Program:** CodePath AI301 — AI Open Source Capstone (Section 1b)
**Status:** Phase II Complete

| | |
|---|---|
| **Project** | [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw) |
| **Issue** | [#9527 — Display error message for invalid hexadecimal color codes](https://github.com/excalidraw/excalidraw/issues/9527) |
| **My fork** | [ATMmonitor667/excalidraw](https://github.com/ATMmonitor667/excalidraw) |
| **Working branch** | [fix/9527-invalid-hex-feedback](https://github.com/ATMmonitor667/excalidraw/tree/fix/9527-invalid-hex-feedback) |
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
3. **Active maintenance.** Excalidraw is a large, actively-maintained project (100k+ stars) with a strong `CONTRIBUTING.md`, so setup and review are well-supported.
4. **Learning goal.** I want to learn how a mature codebase handles input normalization (`normalizeInputColor`) and how it wires user-facing strings through its i18n layer.

From reading the issue, I understand the root cause: `ColorInput` calls `normalizeInputColor(value)`, and when that returns `null` (invalid), the code simply doesn't call `onChange` — so nothing visible happens. My contribution adds an `isValid` state, marks the field `aria-invalid`, and renders an inline error message, improving both the user experience and accessibility.

---

## Phase II — Reproduce & Plan

### Reproduction Process

#### Environment Setup

Excalidraw does not ship a dev container, so I followed the standard README / `CONTRIBUTING.md` setup (the "typical case"):

1. Cloned my fork and checked out my working branch.
2. Confirmed I had **Node ≥ 18** (the version range pinned in `package.json`) and **Yarn 1.22** (the repo's declared `packageManager`).
3. Ran `yarn install` to install dependencies.
4. Started the app with `yarn start`, which runs the Vite dev server for `excalidraw-app` at `http://localhost:3000`.

> _Personalize this: add any specific errors you hit during setup and how you resolved them (e.g. a Node version mismatch fixed with `nvm use`, or a port-in-use conflict). Documenting real friction is exactly what this section rewards._

**Working branch:** https://github.com/ATMmonitor667/excalidraw/tree/fix/9527-invalid-hex-feedback

#### Steps to Reproduce

1. Run `yarn start` and open `http://localhost:3000`.
2. Select a drawing tool (e.g. the rectangle or draw tool) so the left-hand styles panel appears.
3. Click the **stroke** (or background) color swatch to open the color picker.
4. In the hex input field, enter an invalid value — for example `zzzzzz`, `12345`, or `blue`.
5. **Expected:** a clear, visible error message indicating the value is not a valid color.
6. **Actual:** the input is silently ignored — the selected color does not change and no error or feedback is shown.

Reproduced consistently across several invalid inputs (wrong length, non-hex characters, and plain words), confirming it's not a fluke.

### Solution Approach

#### Implementation Plan (UMPIRE)

**Understand.** The hex color input accepts text but gives no feedback when the value can't be parsed as a color. The user can't tell whether their input was rejected or why — a usability and accessibility gap.

**Match.** The same `ColorInput` component already uses React state (`innerValue`), conditional class names via `clsx`, and pulls user-facing strings through the i18n helper `t(...)`. The color-parsing logic already exists in `normalizeInputColor` (`packages/common/src/colors.ts`), which returns `null` for invalid input — I can reuse that return value as my validity signal rather than writing new parsing or regex.

**Plan.**
1. Add an `isValid` boolean state to `ColorInput.tsx`.
2. In `changeColor`, set `isValid` based on whether `normalizeInputColor` returned a value, treating an empty field as not-an-error.
3. Mark the `<input>` `aria-invalid={!isValid}`, and reset validity on blur and when the `color` prop changes.
4. Render an inline error message below the input when the value is invalid.
5. Add styling for the error message / invalid border in `ColorPicker.scss`.
6. Add a `colorPicker.invalidColor` string to `packages/excalidraw/locales/en.json`.

**Files I expect to touch:**
- `packages/excalidraw/components/ColorPicker/ColorInput.tsx`
- `packages/excalidraw/components/ColorPicker/ColorPicker.scss`
- `packages/excalidraw/locales/en.json`

**Implement.** (Phase III) Work happens on the branch above: [fix/9527-invalid-hex-feedback](https://github.com/ATMmonitor667/excalidraw/tree/fix/9527-invalid-hex-feedback).

**Review.** Self-review against the project's `CONTRIBUTING.md`; run `yarn fix` (Prettier + ESLint) and follow Excalidraw's commit-message / PR conventions before opening the PR.

**Evaluate.** Re-run the reproduction steps and confirm an error now appears for invalid input and clears once a valid color is entered; run `yarn test:typecheck` (tsc) and `yarn test` (Vitest) to confirm no regressions; run `yarn test:other` (Prettier check) to confirm formatting.

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
