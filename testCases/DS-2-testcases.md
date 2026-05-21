# Test Plan: Edit Existing Program Details

**Feature:** As an admin user, I want to edit an existing program's details so that I can correct or update program information after creation.

**Scope:** Programs management → Edit program flow (admin role)

**Assumed program fields (based on ACs + common patterns):** Name, Description, Start Date, End Date, Status, plus any other fields visible on the create form.

---

## 1. Positive Flows

### TC-001 — Edit form opens pre-populated with current program data
- **Preconditions:**
  - Logged in as an admin user
  - Program "Web Development 2026" exists with Name="Web Development 2026", Description="Intro course", and other fields populated
  - Admin is on the Programs page
- **Steps:**
  1. Locate the row for "Web Development 2026"
  2. Click the edit (pencil) icon on that row
- **Expected result:**
  - Edit form/modal opens
  - All fields are pre-populated with the program's current values (Name, Description, dates, status, etc.)
  - No field is blank or shows placeholder text
  - Title of the modal indicates Edit mode (e.g., "Edit Program")
- **Priority:** High

### TC-002 — Successfully update program Name and see the change in the list
- **Preconditions:** Editing "Web Development 2026" (TC-001 completed)
- **Steps:**
  1. Clear the Name field
  2. Enter "Web Development 2026 - Updated"
  3. Click Save
- **Expected result:**
  - Modal closes automatically
  - Programs list immediately displays "Web Development 2026 - Updated" in the same row (no manual refresh required)
  - A success confirmation is shown (toast/snackbar, if implemented)
  - The updated value is persisted (reload the page → value remains)
- **Priority:** High

### TC-003 — Editing only the Description preserves all other fields
- **Preconditions:** A program exists with Name="Web Development 2026", Description="Old description", Start Date="2026-01-10", End Date="2026-06-10", Status="Active"
- **Steps:**
  1. Open the program for editing
  2. Change Description to "Updated description text"
  3. Leave all other fields untouched
  4. Click Save
- **Expected result:**
  - Modal closes
  - List/details show Description="Updated description text"
  - Name, Start Date, End Date, and Status remain exactly as before
- **Priority:** High

### TC-004 — Save updates multiple fields in a single submission
- **Preconditions:** Editing an existing program
- **Steps:**
  1. Change Name to "Data Science 2026"
  2. Change Description to "Updated DS program"
  3. Change End Date to "2026-12-31"
  4. Click Save
- **Expected result:** All three updated fields are persisted; untouched fields remain unchanged
- **Priority:** High

### TC-005 — Cancel discards changes
- **Preconditions:** Editing "Web Development 2026"
- **Steps:**
  1. Change Name to "Some new name"
  2. Click Cancel (or close the modal via X / Esc)
- **Expected result:**
  - Modal closes
  - Programs list still shows "Web Development 2026" (original value)
  - No changes persisted on reload
- **Priority:** High

### TC-006 — Save button is disabled until a change is made
- **Preconditions:** Edit form just opened with pre-populated values
- **Steps:**
  1. Do not change anything
  2. Observe the Save button state
- **Expected result:** Save is disabled (or saving a no-op closes the modal without server call) — confirms no accidental writes
- **Priority:** Medium

---

## 2. Negative Flows

### TC-007 — Name cannot be saved as empty
- **Preconditions:** Editing "Web Development 2026"
- **Steps:**
  1. Clear the Name field
  2. Click Save
- **Expected result:**
  - Inline validation error shown on Name field (e.g., "Name is required")
  - Modal does not close
  - No change persisted
- **Priority:** High

### TC-008 — Name cannot be saved as whitespace-only
- **Preconditions:** Editing a program
- **Steps:**
  1. Replace Name with `"   "` (spaces only)
  2. Click Save
- **Expected result:** Validation error ("Name is required" or similar); save blocked
- **Priority:** Medium

### TC-009 — Duplicate program name is rejected
- **Preconditions:** Two programs exist: "Web Development 2026" and "Data Science 2026"
- **Steps:**
  1. Edit "Web Development 2026"
  2. Change Name to "Data Science 2026"
  3. Click Save
- **Expected result:**
  - Error message shown (e.g., "A program with this name already exists")
  - Modal stays open, no data persisted
- **Priority:** High

### TC-010 — End Date earlier than Start Date is rejected
- **Preconditions:** Editing a program with Start Date="2026-03-01"
- **Steps:**
  1. Set End Date to "2026-02-01"
  2. Click Save
- **Expected result:** Inline validation error ("End date must be after start date"); save blocked
- **Priority:** High

### TC-011 — Server error is handled gracefully
- **Preconditions:** Editing a program; back-end returns 500 on save (simulate via network mocking)
- **Steps:**
  1. Change Name to "Web Development 2026 - v2"
  2. Click Save
- **Expected result:**
  - User-friendly error message displayed (e.g., "Couldn't save changes. Please try again.")
  - Modal remains open with the entered values intact
  - List is not updated
- **Priority:** High

### TC-012 — Network loss during save
- **Preconditions:** Editing a program; offline mode enabled mid-save
- **Steps:**
  1. Change Name
  2. Disable network
  3. Click Save
- **Expected result:** Clear error message, no partial UI update, user can retry once network is restored
- **Priority:** Medium

### TC-013 — Non-admin user cannot access edit
- **Preconditions:** Logged in as a non-admin (e.g., teacher/student) role
- **Steps:**
  1. Navigate to Programs page
  2. Attempt to open edit (icon hidden/disabled) or call the edit endpoint directly
- **Expected result:**
  - Edit icon not visible or disabled
  - Direct API call returns 401/403
  - No data modified
- **Priority:** High

### TC-014 — Editing a deleted program shows a clear error
- **Preconditions:** Admin opens edit modal; another admin deletes the same program in another session
- **Steps:**
  1. Make a change
  2. Click Save
- **Expected result:** Error message ("Program no longer exists" or 404 handling); modal closes or shows a recoverable state; list refreshes
- **Priority:** Medium

---

## 3. Edge Cases

### TC-015 — Name at maximum allowed length
- **Preconditions:** Assume Name max length = N (use the documented limit, e.g., 100 chars)
- **Steps:**
  1. Set Name to exactly N characters
  2. Click Save
- **Expected result:** Saves successfully; list shows full value or properly truncated with tooltip
- **Priority:** Medium

### TC-016 — Name exceeding maximum length is rejected/trimmed
- **Preconditions:** Same as TC-015
- **Steps:**
  1. Set Name to N+1 characters
  2. Click Save
- **Expected result:** Either input prevents extra characters or validation error is shown; no oversize value persisted
- **Priority:** Medium

### TC-017 — Special characters in Name
- **Preconditions:** Editing a program
- **Steps:**
  1. Set Name to `Web Dev 2026 — C++/C# & .NET (rev. 2) <admin>`
  2. Click Save
- **Expected result:**
  - Save succeeds (assuming no character restriction)
  - List renders the value exactly (no broken HTML, no XSS execution from `<admin>`)
- **Priority:** High (security-adjacent)

### TC-018 — Unicode / non-Latin Name
- **Preconditions:** Editing a program
- **Steps:**
  1. Set Name to "Веб-розробка 2026 🚀"
  2. Click Save
- **Expected result:** Saves and displays correctly; no encoding issues in the list or detail view
- **Priority:** Medium

### TC-019 — Leading/trailing whitespace in Name
- **Preconditions:** Editing a program
- **Steps:**
  1. Set Name to `"  Web Development 2026  "`
  2. Click Save
- **Expected result:** Either auto-trimmed before save (preferred) or kept as-is consistently — but uniqueness check should treat trimmed value as equivalent to existing names
- **Priority:** Medium

### TC-020 — Same value re-saved (idempotent edit)
- **Preconditions:** Editing "Web Development 2026"
- **Steps:**
  1. Re-type the same Name "Web Development 2026"
  2. Click Save
- **Expected result:** Either Save is disabled (TC-006) or save succeeds without errors; list state unchanged
- **Priority:** Low

### TC-021 — Description with very long text / line breaks
- **Preconditions:** Editing a program
- **Steps:**
  1. Paste a 5,000-character Description with multiple line breaks
  2. Click Save
- **Expected result:** Either accepts up to documented max or shows a clear validation message; rendering in list/details is not broken
- **Priority:** Medium

### TC-022 — Rapid double-click on Save (no duplicate submissions)
- **Preconditions:** Editing a program
- **Steps:**
  1. Change Name
  2. Double-click Save quickly
- **Expected result:** Only one save request fires (button disables on first click); no duplicate side-effects
- **Priority:** Medium

### TC-023 — Concurrent edits by two admins (last-write-wins or conflict)
- **Preconditions:** Two admin sessions open the same program for editing
- **Steps:**
  1. Admin A changes Name to "A version"
  2. Admin B changes Name to "B version"
  3. Both click Save (B saves second)
- **Expected result:** Either last-write-wins with clear behavior, or a conflict/stale-data warning is shown to the later submitter — verify against product spec
- **Priority:** Medium

### TC-024 — Edit reflects in list ordering / search / filters
- **Preconditions:** Programs list sorted alphabetically by Name; filter applied
- **Steps:**
  1. Edit a program and change its Name so it falls into a different sort position or filter bucket
  2. Save
- **Expected result:** List re-sorts/filters correctly without manual refresh
- **Priority:** Medium

### TC-025 — Closing modal via Esc / outside click prompts for unsaved changes
- **Preconditions:** Editing a program with unsaved changes
- **Steps:**
  1. Press Esc or click outside the modal
- **Expected result:** Confirmation prompt before discarding (if implemented); otherwise changes discarded silently and original values remain — behavior should match product spec
- **Priority:** Low

### TC-026 — Browser back/refresh during edit
- **Preconditions:** Edit modal open with unsaved changes
- **Steps:**
  1. Refresh the page
- **Expected result:** Changes are discarded; original data shown in list; no partial state persists
- **Priority:** Low

### TC-027 — Accessibility: keyboard-only edit flow
- **Preconditions:** Admin uses only keyboard
- **Steps:**
  1. Tab to edit icon, press Enter
  2. Tab through fields, edit Name
  3. Tab to Save, press Enter
- **Expected result:** Full flow possible via keyboard; focus is trapped in modal; Esc closes; screen reader announces field labels and errors
- **Priority:** Medium

---

## Ambiguities & Gaps in the ACs

1. **Field list is not enumerated.** ACs reference "Name" and "Description" but say "and other fields remain unchanged" without naming them. The full editable schema (dates, status, capacity, instructor, etc.) needs definition.
2. **Validation rules are missing.** No spec for:
   - Name min/max length
   - Allowed characters / unicode
   - Description max length
   - Date constraints (must end-date be after start-date?)
3. **Uniqueness of program name** is not stated. Should "Web Development 2026" be globally unique, unique per year, or duplicates allowed?
4. **Permissions.** ACs only mention "admin user" — what about other roles? Is the edit icon hidden, disabled, or returns an error?
5. **Modal vs. page.** AC-2 says "the modal closes," implying a modal, but AC-1 says "edit form" — confirm UI pattern.
6. **Success/error feedback.** No mention of toast/snackbar confirmation or how server errors are communicated.
7. **Concurrency.** No spec for what happens when two admins edit the same program simultaneously.
8. **Audit trail.** Should edits be logged (who/when/what changed)? Not in ACs.
9. **Cancel behavior.** No AC covers Cancel/Esc/outside-click — confirm whether a "discard unsaved changes" prompt is required.
10. **Effect of edit on related entities.** If a program has enrolled students or linked courses, does renaming/date-shifting propagate or trigger warnings? Not addressed.
11. **"Immediately shows"** in AC-2 — is this optimistic UI update or post-server confirmation? Important for error-rollback behavior.
12. **Status field semantics.** If Status can be edited (e.g., Active → Archived), are there side effects (hide from enrollment, etc.)?

Recommend clarifying these with PM/design before sign-off so the negative and edge-case tests above can be tightened to exact expected results.
