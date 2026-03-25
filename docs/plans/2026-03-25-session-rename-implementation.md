# Session Rename Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add persistent custom session names to `cc-switch` sessions and expose them in the session manager UI with original-title visibility.

**Architecture:** The backend stores session rename overrides in a dedicated SQLite table and overlays them onto scanned sessions before returning `SessionMeta`. The frontend adds a rename mutation, updates search indexing to include both effective and original titles, and exposes inline rename controls in the detail panel.

**Tech Stack:** Tauri, Rust, rusqlite, React, TanStack Query, Vitest, Testing Library

---

### Task 1: Add backend persistence for session title overrides

**Files:**
- Modify: `src-tauri/src/database/mod.rs`
- Modify: `src-tauri/src/database/schema.rs`
- Modify: `src-tauri/src/database/dao/settings.rs`

**Step 1: Write the failing test**

- Add a unit test that saves a session title override, loads it back, and clears it.

**Step 2: Run test to verify it fails**

Run: Rust test command for the new test.
Expected: FAIL because no session override table/DAO exists.

**Step 3: Write minimal implementation**

- Add `session_overrides` table creation and migration.
- Add DAO helpers to upsert, fetch, list, and clear overrides.

**Step 4: Run test to verify it passes**

Run: Rust test command for the new test.
Expected: PASS.

**Step 5: Commit**

```bash
git add src-tauri/src/database/mod.rs src-tauri/src/database/schema.rs src-tauri/src/database/dao/settings.rs
git commit -m "feat: persist session title overrides"
```

### Task 2: Overlay overrides onto scanned sessions and expose rename command

**Files:**
- Modify: `src-tauri/src/session_manager/mod.rs`
- Modify: `src-tauri/src/commands/session_manager.rs`
- Modify: `src-tauri/src/lib.rs`

**Step 1: Write the failing test**

- Add a unit test that verifies a scanned session receives the custom title while retaining the original title.

**Step 2: Run test to verify it fails**

Run: Rust test command for the new test.
Expected: FAIL because overlay/original title support does not exist.

**Step 3: Write minimal implementation**

- Extend `SessionMeta` with original/custom-title metadata.
- Add overlay logic.
- Add `rename_session` Tauri command and wire it into the invoke handler.

**Step 4: Run test to verify it passes**

Run: Rust test command for the new test.
Expected: PASS.

**Step 5: Commit**

```bash
git add src-tauri/src/session_manager/mod.rs src-tauri/src/commands/session_manager.rs src-tauri/src/lib.rs
git commit -m "feat: expose renamed session metadata"
```

### Task 3: Add frontend API/query support and failing component test

**Files:**
- Modify: `src/types.ts`
- Modify: `src/lib/api/sessions.ts`
- Modify: `src/lib/query/mutations.ts`
- Modify: `tests/msw/state.ts`
- Modify: `tests/msw/handlers.ts`
- Modify: `tests/components/SessionManagerPage.test.tsx`

**Step 1: Write the failing test**

- Add a test covering rename save and restore of original title in the detail pane.

**Step 2: Run test to verify it fails**

Run: `pnpm test:unit tests/components/SessionManagerPage.test.tsx`
Expected: FAIL because rename UI and API are missing.

**Step 3: Write minimal implementation**

- Add rename API types and mutation support.
- Extend test MSW state with rename behavior.

**Step 4: Run test to verify it passes**

Run: `pnpm test:unit tests/components/SessionManagerPage.test.tsx`
Expected: PASS.

**Step 5: Commit**

```bash
git add src/types.ts src/lib/api/sessions.ts src/lib/query/mutations.ts tests/msw/state.ts tests/msw/handlers.ts tests/components/SessionManagerPage.test.tsx
git commit -m "test: cover session rename flow"
```

### Task 4: Add rename UI and search support

**Files:**
- Modify: `src/components/sessions/SessionManagerPage.tsx`
- Modify: `src/components/sessions/utils.ts`
- Modify: `src/hooks/useSessionSearch.ts`
- Modify: `src/i18n/locales/en.json`
- Modify: `src/i18n/locales/zh.json`
- Modify: `src/i18n/locales/ja.json`

**Step 1: Write the failing test**

- Extend the component test so searching by the original title still finds the renamed session.

**Step 2: Run test to verify it fails**

Run: `pnpm test:unit tests/components/SessionManagerPage.test.tsx`
Expected: FAIL because search index does not include original title.

**Step 3: Write minimal implementation**

- Add inline rename controls and original-title display.
- Include `originalTitle` in search indexing and title helpers.
- Add localized labels/messages.

**Step 4: Run test to verify it passes**

Run: `pnpm test:unit tests/components/SessionManagerPage.test.tsx`
Expected: PASS.

**Step 5: Commit**

```bash
git add src/components/sessions/SessionManagerPage.tsx src/components/sessions/utils.ts src/hooks/useSessionSearch.ts src/i18n/locales/en.json src/i18n/locales/zh.json src/i18n/locales/ja.json
git commit -m "feat: add session rename UI"
```

### Task 5: Verify, review, and prepare PR

**Files:**
- Modify: as needed from review feedback

**Step 1: Run verification**

Run the targeted Rust tests, the session manager frontend tests, full unit tests, and typecheck.

**Step 2: Request code review**

- Review the branch diff before PR creation and fix any issues found.

**Step 3: Push branch**

```bash
git push -u origin feat/session-rename-persistence
```

**Step 4: Create PR**

Use `gh pr create` with a concise summary and test plan.

**Step 5: Final verification**

- Re-run the verification commands after any review-driven edits.
