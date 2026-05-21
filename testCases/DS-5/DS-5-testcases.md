# Test Plan: View Programs List

**Feature:** As an admin user, I want to see all programs in a clear list so that I can quickly find and manage them.

**Scope:** Programs page (list view) — rendering of program rows, empty state, performance, accessibility, and how the list integrates with surrounding features (create / edit / delete from DS-2, DS-3, DS-4).

**Assumed surrounding context:**
- Each program row shows at minimum: **Name** and **Description**, plus row-level actions (Edit and Delete icons).
- Admin role is the actor.
- The list is reached via the main navigation (e.g., "Programs" menu item).
- Default behavior unspecified by ACs (sort, pagination, search) is exercised in edge cases and flagged as ambiguities.

---

## 1. Positive Flows

### TC-001 — Programs list displays each program's Name and Description (covers AC1)
- **Preconditions:**
  - Logged in as admin
  - At least 3 programs exist:
    - "Web Development 2026" — Description: "Intro web course"
    - "Data Science 2026" — Description: "Foundations of DS"
    - "Mobile Development 2026" — Description: "iOS + Android basics"
- **Steps:**
  1. Click "Programs" in the main navigation
- **Expected result:**
  - The Programs page loads
  - All three programs are visible as rows
  - Each row shows the correct Name and Description (exact text match)
  - Page title/heading clearly identifies the page (e.g., "Programs")
- **Priority:** High

### TC-002 — Empty state is shown when no programs exist (covers AC2)
- **Preconditions:** Logged in as admin; zero programs in the system
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:**
  - A clear empty-state message is shown (e.g., "No programs yet" or "You haven't created any programs")
  - A visible call-to-action prompts creation (e.g., "Create your first program" button)
  - No table headers, filters, or pagination controls suggesting absent data
  - Clicking the CTA opens the create-program form
- **Priority:** High

### TC-003 — Newly created program appears in the list without manual refresh
- **Preconditions:** Logged in as admin; Programs page open; create-program flow works
- **Steps:**
  1. Click "Create program"
  2. Fill the form with Name = "QA Engineering 2026", Description = "Test plan basics"
  3. Submit the form
- **Expected result:** The new program appears in the list immediately (no full page reload required); success toast shown (if implemented)
- **Priority:** High

### TC-004 — Editing a program updates its row in place (cross-link with DS-2)
- **Preconditions:** Program "Web Development 2026" is visible in the list
- **Steps:**
  1. Edit the program; change Name to "Web Development 2026 - Updated" and Description to "Updated description"
  2. Save
- **Expected result:** The same row now displays the updated Name and Description without a full reload; row position respects current sort
- **Priority:** High

### TC-005 — Deleted program disappears from the list (cross-link with DS-4)
- **Preconditions:** Program "Test Program" is visible in the list
- **Steps:**
  1. Trigger delete on "Test Program" and confirm
- **Expected result:** Row disappears; total count decrements; pagination adjusts; no ghost row remains
- **Priority:** High

### TC-006 — Row actions are present and enabled (Edit and Delete icons)
- **Preconditions:** Logged in as admin; at least one program exists
- **Steps:**
  1. Hover/focus a program row
- **Expected result:**
  - Edit and Delete icons are visible and enabled
  - Icons have accessible labels (e.g., `aria-label="Edit Web Development 2026"`)
- **Priority:** Medium

### TC-007 — Page loads in a reasonable time with a typical dataset
- **Preconditions:** 100 programs exist
- **Steps:**
  1. Navigate to the Programs page; measure time-to-interactive in DevTools
- **Expected result:** Page renders within agreed performance budget (e.g., < 1s on a fast connection, < 3s on slow 3G); no UI freeze; loading state shown if needed
- **Priority:** Medium

---

## 2. Negative Flows

### TC-008 — Non-admin role cannot see the Programs page (or sees only allowed data)
- **Preconditions:** Logged in as a non-admin (e.g., teacher/student) — spec-dependent
- **Steps:**
  1. Navigate to /programs URL directly
- **Expected result:**
  - Either: the route is not exposed in navigation and direct URL returns 403/redirects to a "not authorized" page
  - Or: only data the user is permitted to see is shown (must match the documented authorization model)
- **Priority:** High

### TC-009 — Unauthenticated user is redirected to login
- **Preconditions:** Not logged in
- **Steps:**
  1. Navigate directly to /programs
- **Expected result:** Redirected to the login page; after login, redirected back to /programs
- **Priority:** High

### TC-010 — Server error on list fetch is handled gracefully
- **Preconditions:** Backend returns 500 on the list endpoint (simulate via network mocking)
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:**
  - An error state is shown (e.g., "Couldn't load programs. Please try again.") with a Retry button
  - No partial/garbled UI; no infinite spinner; no stack trace leaked
- **Priority:** High

### TC-011 — Network loss / slow connection
- **Preconditions:** Offline mode enabled or throttled to slow 3G
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:**
  - Loading indicator shown while fetching
  - On full failure: clear offline/error message with retry
  - On slow: list eventually renders without UI corruption
- **Priority:** Medium

### TC-012 — Empty state CTA is hidden for users without create permission
- **Preconditions:** Logged in as a role that can view but not create (spec-dependent); zero programs exist
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:** Empty-state message shown, but the "Create your first program" CTA is hidden or disabled with explanation
- **Priority:** Medium

### TC-013 — XSS payload in Name/Description is rendered as text, not executed
- **Preconditions:** A program exists with Name = `<script>alert(1)</script>` (created via DS-3 happy/edge case) and Description = `"><img src=x onerror=alert(1)>`
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:** Both fields render as literal characters; no JavaScript executes; HTML is not interpreted; no layout breakage
- **Priority:** High (security)

### TC-014 — API returns malformed data — UI degrades gracefully
- **Preconditions:** Backend returns a program with missing Description (null/undefined) or unexpected fields (simulate)
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:**
  - Row still renders for that program with a sensible placeholder (e.g., empty Description or "—")
  - Other rows are unaffected; no whole-page crash
- **Priority:** Medium

---

## 3. Edge Cases

### TC-015 — Single program in the list
- **Preconditions:** Exactly one program exists
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:** The list renders normally with one row; pagination (if shown) does not display useless multi-page controls
- **Priority:** Low

### TC-016 — Large dataset (pagination / virtualization)
- **Preconditions:** 1,000 programs exist
- **Steps:**
  1. Navigate to the Programs page
  2. Scroll through / paginate
- **Expected result:**
  - Either pagination is shown (with page size and navigation controls) or the list virtualizes/lazy-loads
  - No browser freeze; smooth scrolling; total count visible
  - Sort/filter (if present) operate correctly across pages
- **Priority:** Medium

### TC-017 — Very long Name and Description text wrap or truncate cleanly
- **Preconditions:** A program exists with a max-length Name (e.g., 100 chars) and a 5,000-character Description
- **Steps:**
  1. View it in the list
- **Expected result:**
  - Layout doesn't break (no horizontal scroll, no overlapping cells)
  - Long text truncates with ellipsis; full text available via tooltip or by clicking into details
- **Priority:** Medium

### TC-018 — Special characters and Unicode render correctly
- **Preconditions:** Programs with names containing: `Informatique & IA — Niveau 2`, `Développement Web 🚀`, `تطوير الويب 2026`
- **Steps:**
  1. View the Programs list
- **Expected result:** All characters render as entered (no `&amp;` artifacts, no mojibake); RTL text is rendered with correct direction; emoji appears correctly
- **Priority:** Medium

### TC-019 — Default sort order is consistent
- **Preconditions:** 5 programs exist with varied names and creation dates
- **Steps:**
  1. Reload the Programs page several times
- **Expected result:** Default sort order (e.g., newest first, or alphabetical) is deterministic and documented; same order on every reload (no random shuffling)
- **Priority:** Medium

### TC-020 — Sorting (if available) reorders rows correctly
- **Preconditions:** Sort controls exist (e.g., click column header)
- **Steps:**
  1. Sort by Name ascending → descending
  2. Sort by Description / Created date if available
- **Expected result:** Rows reorder accordingly; sort indicator (arrow) reflects the active column; secondary sort behavior is documented (e.g., tie-break by id)
- **Priority:** Medium

### TC-021 — Search / filter (if available) narrows the list
- **Preconditions:** Search input exists; programs exist with names containing "Web", "Data", "Mobile"
- **Steps:**
  1. Type "Web" in the search box
- **Expected result:**
  - List shows only matching programs
  - Total count reflects the filtered result
  - Clearing the search restores the full list
  - Empty search result shows a "No matches" state, not the global empty state
- **Priority:** Medium

### TC-022 — Pagination boundary navigation
- **Preconditions:** Enough programs for at least 3 pages
- **Steps:**
  1. Navigate to last page; verify the link to "next" is disabled
  2. Navigate back to first page; verify "previous" is disabled
- **Expected result:** Boundary controls are disabled at the correct positions; clicking them does nothing; no JS errors
- **Priority:** Low

### TC-023 — Empty state after deleting the last program
- **Preconditions:** One program exists
- **Steps:**
  1. Delete it (cross-link DS-4)
- **Expected result:** The list transitions to the empty state immediately (same content as TC-002), with the "Create your first program" CTA visible
- **Priority:** Medium

### TC-024 — Keyboard-only navigation works (a11y)
- **Preconditions:** Admin uses keyboard only
- **Steps:**
  1. Tab from the page heading through the list rows and their action icons
- **Expected result:**
  - Focus order is logical (top to bottom, left to right)
  - Focused row / icon has a visible focus indicator
  - Enter on a row's primary action triggers edit (or whatever the documented behavior is)
- **Priority:** Medium

### TC-025 — Screen reader announces list semantics
- **Preconditions:** Admin uses VoiceOver / NVDA
- **Steps:**
  1. Navigate to the list and traverse rows
- **Expected result:**
  - Semantic markup is used (`<table>` with `<th scope="col">`, or `<ul>` with proper roles)
  - Each row's Name and Description are announced
  - Action buttons have accessible names (e.g., "Edit Web Development 2026", "Delete Web Development 2026")
  - Empty state is announced as a status message
- **Priority:** Medium

### TC-026 — Responsive layout (mobile / narrow viewport)
- **Preconditions:** Viewport widths: 1440px (desktop), 768px (tablet), 375px (mobile)
- **Steps:**
  1. View the Programs page at each width
- **Expected result:** Layout adapts; no horizontal scrolling; Description may collapse to a second line or be hidden behind an expand toggle; row actions remain reachable
- **Priority:** Medium

### TC-027 — Concurrent updates: another admin creates a program while I'm viewing the list
- **Preconditions:** Two admin sessions
- **Steps:**
  1. Admin A: viewing the Programs page
  2. Admin B: creates a new program
  3. Admin A: refreshes or waits for auto-refresh (if implemented)
- **Expected result:** The new program eventually appears for Admin A — either on refresh, or via real-time/polling update if the spec calls for it
- **Priority:** Low

### TC-028 — Empty Description renders cleanly
- **Preconditions:** A program exists with Name = "MVP 2026" and Description = ""
- **Steps:**
  1. View the list
- **Expected result:** Row shows the Name; Description column is empty or shows a neutral placeholder (e.g., "—"); layout is consistent with rows that have descriptions
- **Priority:** Low

### TC-029 — Loading skeleton / spinner is shown during initial fetch
- **Preconditions:** Slow network (throttled)
- **Steps:**
  1. Navigate to the Programs page
- **Expected result:** A loading indicator (spinner or skeleton rows) is shown until data arrives; it disappears once the list (or empty state) is ready; never shown together with the final list
- **Priority:** Medium

### TC-030 — Direct deep link / bookmark works
- **Preconditions:** User bookmarks /programs while filters/sort are applied (e.g., `/programs?sort=name&dir=asc&q=Web`)
- **Steps:**
  1. Open the bookmark in a new tab
- **Expected result:** Same filtered/sorted view loads (if URL state is supported); if not supported, the unfiltered list loads — must match documented behavior
- **Priority:** Low

### TC-031 — Browser refresh preserves user-visible state appropriately
- **Preconditions:** List with a search applied
- **Steps:**
  1. Press F5 / Cmd+R
- **Expected result:** If URL state holds the search, the search persists; if not, it resets. Either is acceptable but must match spec and be consistent.
- **Priority:** Low

### TC-032 — Right-click / context menu and copy work on row text
- **Preconditions:** A program row with Name and Description
- **Steps:**
  1. Select and copy the Name text
- **Expected result:** Selecting and copying works; clipboard contains the literal program name (no extra markup)
- **Priority:** Low

---

## Ambiguities & Gaps in the ACs

1. **Other columns / fields.** ACs only specify Name and Description. Should Status, Start/End Date, Created date, owner, enrolled count, etc. also be shown?
2. **Sort order.** Default sort is unspecified — newest first? alphabetical? Are column sorts available?
3. **Search / filter.** Not mentioned in ACs but expected at non-trivial dataset sizes; need to define scope, fields searched, and case sensitivity.
4. **Pagination / virtualization.** Behavior for 100+, 1,000+ programs is undefined.
5. **Permissions / roles.** Only "admin user" is mentioned; behavior for other roles (visibility, ability to create from empty state) is unclear.
6. **Empty state CTA target.** Confirm the CTA opens the create form and respects role-based permissions.
7. **Real-time updates.** Should the list reflect creates/edits/deletes from other admins without manual refresh?
8. **Performance budget.** No SLA/SLO defined for list load time.
9. **Long-text handling.** Truncation rules for Name and Description are not specified.
10. **Accessibility requirements.** Keyboard, screen reader, focus order, and color-contrast requirements not in scope of these ACs but typically required.
11. **Responsive design.** Mobile/tablet expectations not stated.
12. **URL state.** Whether filters/sort/page are reflected in the URL (for shareable links / refresh) is undefined.
13. **Action icon discoverability.** Whether edit/delete icons are always visible or only on hover/focus — UX choice with a11y implications.
14. **Localization.** If the app supports multiple locales, Name/Description are stored in one language only? Translations? Date formats?
15. **Cross-link consistency.** The list must reflect changes from DS-2 (edit), DS-3 (validation), DS-4 (delete) immediately and consistently — explicit confirmation needed.

Recommend confirming items 1, 2, 3, 4, 7, and 12 with PM/design before sign-off — they materially shape both the implementation and the test cases above.
