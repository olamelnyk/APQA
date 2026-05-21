# Test Plan: Delete Program with Confirmation

**Feature:** As an admin user, I want to delete a program I no longer need, with a confirmation step to prevent accidental deletion.

**Scope:** Programs list → Delete icon → Confirmation dialog → Server delete → List refresh.

**Assumed surrounding context:**
- Delete is triggered by a trash/delete icon on each row of the Programs list (and potentially the program details page).
- A modal confirmation dialog is shown before any destructive action.
- Admin role is the actor.
- Test data: at least one program "Test Program" and one program "Web Development 2026" exist (the latter is used elsewhere in DS-2/DS-3).

---

## 1. Positive Flows

### TC-001 — Confirmation dialog opens when the delete icon is clicked
- **Preconditions:**
  - Logged in as admin
  - On the Programs page
  - Program "Test Program" exists
- **Steps:**
  1. Locate the row for "Test Program"
  2. Click the delete (trash) icon on that row
- **Expected result:**
  - A confirmation dialog opens
  - Dialog clearly identifies the target program by name (e.g., 'Delete "Test Program"?')
  - Dialog contains a destructive primary action (e.g., "Delete") and a non-destructive secondary action (e.g., "Cancel")
  - Focus moves to the dialog; background list is non-interactive (overlay)
- **Priority:** High

### TC-002 — Program is removed from the list after confirmation (covers AC1)
- **Preconditions:** Confirmation dialog for "Test Program" is open (TC-001 completed)
- **Steps:**
  1. Click the Delete (confirm) button in the dialog
- **Expected result:**
  - Dialog closes
  - "Test Program" no longer appears in the Programs list (no manual refresh required)
  - Success confirmation shown (toast/snackbar, if implemented)
  - On page reload, "Test Program" is still gone (persistence verified)
- **Priority:** High

### TC-003 — Cancel keeps the program in the list (covers AC2)
- **Preconditions:**
  - Logged in as admin
  - Program "Test Program" exists
  - Confirmation dialog is open
- **Steps:**
  1. Click the Cancel button in the dialog
- **Expected result:**
  - Dialog closes
  - "Test Program" remains in the Programs list, unchanged
  - No DELETE request is sent to the server (verified via network panel)
- **Priority:** High

### TC-004 — Closing the dialog via X / Esc / overlay click also cancels the delete
- **Preconditions:** Confirmation dialog is open for "Test Program"
- **Steps:**
  1. Close the dialog by pressing Esc (repeat with the X button and an outside/overlay click)
- **Expected result:** Dialog closes, no delete happens, program remains in the list, no DELETE request fired
- **Priority:** Medium

### TC-005 — Deletion updates total count, sort order and pagination
- **Preconditions:**
  - Programs list sorted alphabetically by Name
  - List shows a total count (e.g., "12 programs")
  - Pagination has e.g. 1 item on page 2
- **Steps:**
  1. Delete that one item on page 2
- **Expected result:**
  - Total count decrements (e.g., "11 programs")
  - Pagination adjusts (e.g., user lands on page 1 or last page with content; no empty page 2 stuck in view)
  - Sort/filter state is preserved
- **Priority:** Medium

### TC-006 — Success feedback is shown after deletion
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Delete "Test Program" via the confirmation flow
- **Expected result:** A success message is displayed (e.g., toast "Program 'Test Program' deleted") and disappears after a short delay; message is dismissable
- **Priority:** Low

### TC-007 — Delete from the program details page works the same as the list (if available)
- **Preconditions:** A details/edit page exists with a delete button
- **Steps:**
  1. Open the details page for "Test Program"
  2. Click Delete
  3. Confirm
- **Expected result:** Same confirmation flow; on success, user is navigated back to the Programs list and "Test Program" is gone
- **Priority:** Medium

---

## 2. Negative Flows

### TC-008 — Non-admin user cannot delete a program
- **Preconditions:** Logged in as a non-admin role (e.g., teacher/student) or unauthenticated
- **Steps:**
  1. Navigate to the Programs page
  2. Attempt to access the delete icon (or call the DELETE endpoint directly)
- **Expected result:**
  - Delete icon is hidden or disabled in the UI
  - Direct API call returns 401/403
  - No program is deleted
- **Priority:** High

### TC-009 — Server error on delete is handled gracefully
- **Preconditions:** Backend returns 500 when DELETE is called (simulate via network mocking)
- **Steps:**
  1. Open the confirmation dialog for "Test Program"
  2. Click Delete
- **Expected result:**
  - Friendly error shown (e.g., "Couldn't delete program. Please try again.")
  - Program remains in the list
  - Confirmation dialog either stays open with a retry button or closes with the error toast — must be consistent with product spec
- **Priority:** High

### TC-010 — Network loss during delete does not corrupt list state
- **Preconditions:** Confirmation dialog open
- **Steps:**
  1. Disable network
  2. Click Delete
- **Expected result:** Clear error message; program is NOT optimistically removed from the list (or, if optimistically removed, it is restored on failure)
- **Priority:** Medium

### TC-011 — Concurrent deletion: program already deleted in another session
- **Preconditions:** Admin A opens the confirmation dialog for "Test Program"; Admin B deletes the same program in another session
- **Steps:**
  1. Admin A clicks Delete
- **Expected result:**
  - Server returns 404 (or equivalent "already deleted")
  - User-friendly message ("Program no longer exists" or "Already deleted")
  - List refreshes to reflect the current state
- **Priority:** Medium

### TC-012 — Program with active dependencies cannot be deleted (spec-dependent)
- **Preconditions:** A program "Web Development 2026" has enrolled students / linked courses / active sessions
- **Steps:**
  1. Click delete icon for "Web Development 2026"
  2. Confirm
- **Expected result:** Either:
  - Server rejects with a 409/422 and a clear message ("Cannot delete: program has X enrolled students. Archive instead."), OR
  - Confirmation dialog shows a warning with the dependency count before allowing delete
  - Program is NOT deleted; dependencies remain intact
- **Priority:** High
- **Note:** Behavior depends on product spec (block, cascade, or archive). Confirm with PM.

### TC-013 — DELETE endpoint cannot be called without authentication
- **Preconditions:** API access without a valid session token
- **Steps:**
  1. Send `DELETE /programs/<id>` without (or with an invalid) auth token
- **Expected result:** 401 Unauthorized; no program deleted
- **Priority:** High

### TC-014 — DELETE on a non-existent program id returns 404
- **Preconditions:** Authenticated admin
- **Steps:**
  1. Send `DELETE /programs/<non-existent-id>`
- **Expected result:** 404 Not Found; no side effects; no error stack trace leaked to the client
- **Priority:** Medium

### TC-015 — Cancel does not fire any server request
- **Preconditions:** Confirmation dialog open with network panel recording
- **Steps:**
  1. Click Cancel
- **Expected result:** Zero DELETE requests sent; only the dialog dismiss is observed
- **Priority:** Medium

### TC-016 — Delete via direct URL after deletion: program detail page is no longer accessible
- **Preconditions:** "Test Program" has just been deleted
- **Steps:**
  1. Navigate directly to the program's details URL (e.g., bookmarked link)
- **Expected result:** 404 (or a friendly "Not found" page); no leftover cached data shown
- **Priority:** Medium

---

## 3. Edge Cases

### TC-017 — Rapid double-click on Delete does not delete twice
- **Preconditions:** Confirmation dialog open
- **Steps:**
  1. Double-click the Delete button quickly
- **Expected result:** Only one DELETE request is sent (button disables on first click and shows a loading state); no duplicate side effects, no error from second click
- **Priority:** Medium

### TC-018 — Delete dialog correctly identifies the program when names contain special characters / Unicode
- **Preconditions:** A program named "Informatique & IA — Niveau 2 🚀" exists
- **Steps:**
  1. Click the delete icon for that program
- **Expected result:** Dialog shows the program name exactly as stored (no broken HTML, no `&amp;` encoding artifacts, no truncation hiding identifying parts)
- **Priority:** Medium

### TC-019 — Delete confirmation matches the correct row when multiple programs share similar names
- **Preconditions:** Two programs exist: "Web Development 2026" and "Web Development 2026 - Updated"
- **Steps:**
  1. Click the delete icon on the row for "Web Development 2026 - Updated"
  2. Confirm
- **Expected result:** Only "Web Development 2026 - Updated" is deleted; "Web Development 2026" still exists
- **Priority:** High (regression risk)

### TC-020 — Deleting the last program shows an appropriate empty state
- **Preconditions:** Only one program exists
- **Steps:**
  1. Delete it via the confirmation flow
- **Expected result:** List shows an empty state ("No programs yet" or equivalent), with a clear call to action (e.g., "Create program"); no UI breakage
- **Priority:** Low

### TC-021 — Filter / search / sort state is preserved after delete
- **Preconditions:** Programs list filtered by Status="Active" and searched for "Web"
- **Steps:**
  1. Delete one filtered program
- **Expected result:** Filter, search and sort remain applied; only the deleted row disappears
- **Priority:** Medium

### TC-022 — Browser refresh while confirmation dialog is open
- **Preconditions:** Confirmation dialog open
- **Steps:**
  1. Refresh the browser tab
- **Expected result:** Dialog state is not persisted; program is not deleted; list shows current state
- **Priority:** Low

### TC-023 — Keyboard-only deletion flow is fully usable (a11y)
- **Preconditions:** Admin uses keyboard only
- **Steps:**
  1. Tab to the delete icon, press Enter
  2. Focus moves into the dialog (focus is trapped inside)
  3. Tab to Delete, press Enter
- **Expected result:**
  - Dialog opens and traps focus
  - Esc cancels; Enter on the focused primary button confirms
  - On close, focus returns to a sensible element (the delete icon row or the next focusable row)
- **Priority:** Medium

### TC-024 — Screen reader announces the dialog and the destructive action
- **Preconditions:** Admin uses VoiceOver / NVDA
- **Steps:**
  1. Trigger the delete flow
- **Expected result:** Dialog has `role="dialog"` (or `alertdialog`), aria-labelledby/aria-describedby announcing the title and the impact of confirming; primary button is announced as "Delete, button"
- **Priority:** Medium

### TC-025 — Re-creating a program with the same name after deletion is allowed (spec-dependent)
- **Preconditions:** "Test Program" was just deleted
- **Steps:**
  1. Create a new program with Name="Test Program"
- **Expected result:** Creation succeeds (assuming hard delete or that soft-deleted names are released). If soft-delete keeps the name reserved, expect a duplicate error — must match documented behavior. Cross-link with DS-3 TC-011.
- **Priority:** Medium

### TC-026 — Audit trail records the deletion (if implemented)
- **Preconditions:** Audit logging is enabled
- **Steps:**
  1. Delete "Test Program"
  2. Inspect audit log (admin UI or API)
- **Expected result:** Log entry contains: actor (admin id/email), action (DELETE), target (program id and name snapshot), timestamp; entry is immutable
- **Priority:** Low

### TC-027 — Optimistic UI rollback on server failure
- **Preconditions:** UI uses optimistic updates; server returns 500
- **Steps:**
  1. Delete "Test Program"
- **Expected result:** Row visually disappears, then reappears within ~1s with an error toast — no inconsistent state
- **Priority:** Medium

### TC-028 — Deleting multiple programs in quick succession is consistent
- **Preconditions:** At least 3 programs exist
- **Steps:**
  1. Delete program A → confirm
  2. Immediately delete program B → confirm
  3. Immediately delete program C → confirm
- **Expected result:** All three are deleted; list count decreases by 3; no UI glitches, no leftover dialogs stacked
- **Priority:** Medium

### TC-029 — Bulk delete (if supported) requires confirmation per the same rules
- **Preconditions:** Bulk-select UI exists (checkboxes + "Delete selected" button)
- **Steps:**
  1. Select 3 programs
  2. Click "Delete selected"
- **Expected result:** Single confirmation dialog lists all selected program names; on confirm, all 3 are deleted; cancel keeps all 3
- **Priority:** Low (only if bulk delete is in scope)

### TC-030 — Delete button is hidden / disabled while another delete is in flight
- **Preconditions:** Slow network; an earlier delete is still pending
- **Steps:**
  1. Try to trigger another delete while the previous request hasn't completed
- **Expected result:** UI prevents the second action OR queues it safely; no duplicate confirmations stacked; no race condition between requests
- **Priority:** Low

### TC-031 — Confirmation dialog wording is destructive-action-clear
- **Preconditions:** Visual/UX review
- **Steps:**
  1. Open the delete confirmation dialog
- **Expected result:**
  - Primary button label is "Delete" (not generic "OK")
  - Primary button uses a destructive style (red/danger)
  - Body text states that the action is permanent (or "moves to trash" if soft delete) — wording matches what actually happens
- **Priority:** Medium

---

## Ambiguities & Gaps in the ACs

1. **Hard vs soft delete.** ACs say "removed from the program list" — does this also delete the underlying record, or archive/soft-delete it? Determines TC-016, TC-025, and dependency handling.
2. **Dependent records.** What happens to enrolled students, courses, schedules, or other entities linked to the program? Block / cascade / orphan / require archive first? (TC-012)
3. **Permissions.** Only "admin" is mentioned. Are there sub-roles (e.g., super-admin only can delete)? Any per-program ownership rules?
4. **Audit trail.** Should deletions be logged for compliance? (TC-026)
5. **Undo / restore.** Is there an undo window (e.g., "Undo" in the success toast) or a restore-from-trash feature?
6. **Bulk delete.** Out of scope or implicit? (TC-029)
7. **Confirmation strength.** Simple click-to-confirm, or "type the program name to confirm" pattern (common for high-impact deletes)?
8. **Concurrent deletes.** Behavior when two admins delete the same record simultaneously is not specified. (TC-011)
9. **Success/error feedback.** No mention of toasts, inline messages, or animation. (TC-006)
10. **Optimistic update behavior.** Should the row disappear instantly, or only after server confirmation? Important for failure rollback. (TC-027)
11. **Accessibility requirements.** Keyboard, focus trap, ARIA roles not specified. (TC-023, TC-024)
12. **Delete from details page.** AC only mentions a delete icon on the list; unclear if there's also a delete button on the details/edit screen. (TC-007)
13. **Validation on re-creating same name post-delete.** Cross-cuts DS-3 (uniqueness). (TC-025)
14. **Rate limiting / abuse.** No mention of how many deletes per minute an admin can perform.

Recommend clarifying items 1, 2, 5, and 7 with PM/design before sign-off — they materially change expected results across multiple test cases.
