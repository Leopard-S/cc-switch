# Session Rename Design

**Goal:** Add persistent custom session names to `cc-switch` session management without modifying original session files.

## Requirements

- Users can rename a session from the real `cc-switch` session manager UI.
- The rename persists across app restarts.
- The original session title remains available in the detail view.
- Search matches both the custom name and the original/originally scanned metadata.
- Scope is limited to Windows support for now, but the storage model should remain cross-platform-safe.

## Options Considered

### 1. Persist custom names in frontend local storage

- Lowest implementation cost.
- Weak fit for a desktop app that already owns a database.
- Harder to keep consistent with backend-derived session data.

### 2. Persist custom names as JSON in the `settings` key-value table

- Small backend diff.
- Works for persistence.
- Becomes awkward when session metadata grows beyond one field.

### 3. Persist custom names in a dedicated `session_overrides` table

- Clean schema for session-specific metadata.
- Easy to extend later with notes, tags, pinning, and timestamps.
- Requires one schema migration and a small DAO surface.

## Chosen Design

Use a dedicated `session_overrides` SQLite table in `cc-switch.db`, keyed by `provider_id + session_id + source_path`.

The backend session scan keeps reading provider-native session files exactly as it does today. After scan, `cc-switch` overlays any stored custom title onto the returned `SessionMeta` and preserves the scanned title in a new `original_title` field.

## Data Model

New table:

- `provider_id TEXT NOT NULL`
- `session_id TEXT NOT NULL`
- `source_path TEXT NOT NULL`
- `custom_title TEXT NOT NULL`
- `updated_at INTEGER NOT NULL`
- primary key on `(provider_id, session_id, source_path)`

New `SessionMeta` fields:

- `original_title?: string`
- `has_custom_title?: boolean`

Behavior:

- No override: `title` remains the scanned title and `original_title` is omitted.
- Override present: `title` becomes the custom title, `original_title` keeps the scanned title, `has_custom_title` is `true`.
- Clearing a rename deletes the override row and reverts to the scanned title.

## UI

Add rename controls to the right-side session detail header.

- Default state shows the current display title.
- Clicking rename enters inline edit mode.
- Save writes the custom title.
- Clear/remove rename restores the original title.
- When a session has a custom title, the detail panel also shows the original title underneath.

The left session list continues to show the effective display title only.

## Search

The frontend search index will include:

- `title` (effective display title)
- `originalTitle`
- `summary`
- `projectDir`
- `sourcePath`
- `sessionId`

This preserves discoverability after a rename.

## Testing

Backend:

- DAO tests for save/load/clear override behavior.
- Session overlay tests for preserving original title and effective title.

Frontend:

- Session manager test for rename flow and original title display.
- Search test ensuring old/original title still matches after rename.
