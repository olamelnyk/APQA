# Test Plan: Prevent Invalid or Duplicate Program Names

**Feature:** As an admin user, I want the system to prevent invalid or duplicate program names so that data integrity is maintained.

**Scope:** Program creation form → Name field validation (whitespace handling, allowed characters, uniqueness) at both client and server layers.

**Assumed surrounding context:**
- "Program" entity has at minimum: `Name`, plus other required fields (e.g., Description, Start Date, End Date).
- Admin role is the actor; the form is the Create Program modal/page used in DS-2.
- Validation runs on Submit (and may also run on blur/while typing).

---

## 1. Positive Flows

### TC-001 — Standard alphanumeric program name is accepted
- **Preconditions:**
  - Logged in as admin
  - On the Program creation form
  - No program named "Mobile Development 2026" exists
- **Steps:**
  1. Enter Name = "Mobile Development 2026"
  2. Fill all other required fields with valid values
  3. Click Create
- **Expected result:**
  - Form submits successfully
  - New program "Mobile Development 2026" appears in the Programs list
  - Success confirmation shown (toast/snackbar, if implemented)
- **Priority:** High

### TC-002 — Name with allowed special characters is accepted (covers AC2)
- **Preconditions:**
  - Logged in as admin
  - On the Program creation form
  - No program named "Informatique & IA - Niveau 2" exists
- **Steps:**
  1. Enter Name = `Informatique & IA - Niveau 2`
  2. Fill all other required fields
  3. Click Create
- **Expected result:**
  - Program is created
  - Name is stored and displayed exactly as entered, including `&` and `-`
  - No HTML-encoding artifacts (e.g., `&amp;`) shown in the list
- **Priority:** High

### TC-003 — Name with accented / Unicode characters is accepted
- **Preconditions:** Logged in as admin, on the creation form
- **Steps:**
  1. Enter Name = "Développement Web — Niveau Avancé"
  2. Fill other required fields
  3. Click Create
- **Expected result:** Program created; name rendered correctly in list/details (no mojibake)
- **Priority:** Medium

### TC-004 — Leading/trailing whitespace is trimmed before save
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = `"   Web Development 2027   "` (spaces around real content)
  2. Fill other required fields
  3. Click Create
- **Expected result:**
  - Program is created
  - Stored name is "Web Development 2027" (whitespace trimmed)
  - Uniqueness check treats it as equal to an existing "Web Development 2027"
- **Priority:** High

### TC-005 — Name at minimum allowed length is accepted
- **Preconditions:** Documented Name min length = M (e.g., 2 characters)
- **Steps:**
  1. Enter Name with exactly M characters (e.g., "Go")
  2. Fill other required fields
  3. Click Create
- **Expected result:** Program is created successfully
- **Priority:** Medium

### TC-006 — Name at maximum allowed length is accepted
- **Preconditions:** Documented Name max length = N (e.g., 100 characters)
- **Steps:**
  1. Enter Name with exactly N characters
  2. Fill other required fields
  3. Click Create
- **Expected result:** Program is created; list displays the full name (or truncates with a tooltip showing full text)
- **Priority:** Medium

### TC-007 — Same name allowed in different case if uniqueness is case-sensitive (spec-dependent)
- **Preconditions:**
  - A program "Web Development 2026" exists
  - Product spec: uniqueness is **case-sensitive**
- **Steps:**
  1. Enter Name = "web development 2026" (different case)
  2. Click Create
- **Expected result:** Program is created (case-sensitive uniqueness)
- **Priority:** Medium
- **Note:** If spec says case-insensitive uniqueness, this test should be in Negative Flows instead — see TC-013.

---

## 2. Negative Flows

### TC-008 — Whitespace-only name is rejected (covers AC1)
- **Preconditions:** Logged in as admin, on the creation form
- **Steps:**
  1. Enter Name = `"   "` (three spaces)
  2. Fill other required fields
  3. Click Create
- **Expected result:**
  - Form is **not** submitted
  - Inline validation error shown on the Name field (e.g., "Name is required")
  - No network request is sent (or server returns 400 if the client allows it)
  - No program is created
- **Priority:** High

### TC-009 — Empty name is rejected
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Leave Name blank
  2. Fill other required fields
  3. Click Create
- **Expected result:** Inline error "Name is required"; submission blocked; no program created
- **Priority:** High

### TC-010 — Tab/newline-only name is rejected
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = `"\t\n"` (tab + newline, or paste invisible whitespace)
  2. Click Create
- **Expected result:** Treated same as whitespace-only — validation error, no submission
- **Priority:** Medium

### TC-011 — Exact-duplicate name is rejected (covers AC3)
- **Preconditions:** A program "Web Development 2026" already exists
- **Steps:**
  1. Enter Name = "Web Development 2026"
  2. Fill other required fields
  3. Click Create
- **Expected result:**
  - Error message shown (e.g., "A program with this name already exists")
  - Form remains open, focus on Name field, all entered data preserved
  - No new program is created (verified by list count)
- **Priority:** High

### TC-012 — Duplicate name with extra whitespace is rejected (trim + uniqueness)
- **Preconditions:** "Web Development 2026" exists
- **Steps:**
  1. Enter Name = `"  Web Development 2026  "`
  2. Click Create
- **Expected result:** Treated as duplicate after trim → rejected with the same error as TC-011
- **Priority:** High

### TC-013 — Case-insensitive duplicate is rejected (if spec says case-insensitive)
- **Preconditions:**
  - "Web Development 2026" exists
  - Product spec: uniqueness is **case-insensitive**
- **Steps:**
  1. Enter Name = "WEB DEVELOPMENT 2026"
  2. Click Create
- **Expected result:** Rejected as duplicate
- **Priority:** High
- **Note:** Mutually exclusive with TC-007 — confirm spec.

### TC-014 — Name exceeding max length is rejected
- **Preconditions:** Documented Name max = N
- **Steps:**
  1. Enter Name with N+1 characters (paste, since the input may cap typed characters)
  2. Click Create
- **Expected result:** Either input prevents extra characters with a visible counter, or validation error "Name must be at most N characters"; no program created
- **Priority:** Medium

### TC-015 — Name shorter than min length is rejected
- **Preconditions:** Documented Name min = M (e.g., 2)
- **Steps:**
  1. Enter Name = "A" (M-1 characters)
  2. Click Create
- **Expected result:** Inline error "Name must be at least M characters"; submission blocked
- **Priority:** Medium

### TC-016 — Server-side validation rejects bypassed client validation
- **Preconditions:** Send `POST /programs` directly (e.g., via curl/Postman) with Name = `"   "` or with a duplicate name
- **Steps:**
  1. Submit request directly to API with invalid name
- **Expected result:** Server returns 4xx (400/409) with a clear error code/message; no program persisted
- **Priority:** High (data integrity safety net)

### TC-017 — Non-admin role cannot create programs (and thus cannot bypass validation)
- **Preconditions:** Logged in as non-admin (e.g., teacher/student) or unauthenticated
- **Steps:**
  1. Attempt to access the creation form / call the create endpoint
- **Expected result:** UI doesn't expose the form, or API returns 401/403; no program created
- **Priority:** High

### TC-018 — Duplicate-check race condition (concurrent creation)
- **Preconditions:** No program "Concurrent 2026" exists; two admin sessions open the form simultaneously
- **Steps:**
  1. Admin A enters "Concurrent 2026" and clicks Create
  2. Admin B enters "Concurrent 2026" and clicks Create at the same moment
- **Expected result:** Exactly one program is created; the other request gets the duplicate error (server-side unique constraint must enforce this — client-side check alone is not enough)
- **Priority:** High

### TC-019 — Network/server error during create is handled
- **Preconditions:** Backend returns 500 on submit (simulate with network mocking)
- **Steps:**
  1. Enter a valid, unique name and other fields
  2. Click Create
- **Expected result:**
  - Friendly error message displayed ("Couldn't create program. Please try again.")
  - Form data preserved, modal remains open
  - No duplicate retry side effects (idempotent UI)
- **Priority:** High

---

## 3. Edge Cases

### TC-020 — Internal multiple spaces are preserved (not collapsed)
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = "Web    Development 2026" (multiple inner spaces)
  2. Click Create
- **Expected result:**
  - Either the system stores it exactly as entered, OR collapses internal whitespace consistently — behavior must be documented
  - Whichever behavior, uniqueness check must be consistent (so this is or isn't a duplicate of "Web Development 2026")
- **Priority:** Medium

### TC-021 — HTML/script-like characters are stored but not executed (XSS check)
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = `<script>alert(1)</script>`
  2. Fill other fields
  3. Click Create
- **Expected result:**
  - Program is created (assuming `<` `>` are not in a blocklist), OR rejected with a clear validation message
  - In list/detail views the text is rendered as literal characters; no JavaScript executes; no DOM is broken
- **Priority:** High (security)

### TC-022 — SQL/NoSQL injection–style input is stored safely
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = `Robert'); DROP TABLE programs;--`
  2. Click Create
- **Expected result:** Either accepted as a literal string and saved (no DB damage), or rejected by validation; DB remains intact
- **Priority:** High (security)

### TC-023 — Emoji and 4-byte Unicode are accepted
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = "Data Science 🚀 2026"
  2. Click Create
- **Expected result:** Saved and rendered correctly everywhere (list, detail, search). No encoding errors. If DB column doesn't support 4-byte UTF-8, validation should reject with a clear message rather than silently corrupt.
- **Priority:** Medium

### TC-024 — Right-to-left language input is rendered correctly
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = "تطوير الويب 2026"
  2. Click Create
- **Expected result:** Saved; rendered with correct RTL direction in all views
- **Priority:** Low

### TC-025 — Mixed-script + accents + punctuation
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Enter Name = "C++ & .NET — Niveau Pro (Été 2026)"
  2. Click Create
- **Expected result:** Accepted; rendered exactly as entered; no broken HTML, no truncation
- **Priority:** Medium

### TC-026 — Zero-width / homoglyph characters
- **Preconditions:** "Web Development 2026" exists
- **Steps:**
  1. Enter Name = "Web Development 2026" but with a Cyrillic "е" replacing one Latin "e", OR with a zero-width space appended
  2. Click Create
- **Expected result:** Depending on policy:
  - Treated as visually duplicate → rejected (preferred for integrity), OR
  - Accepted as a different string → documented behavior with a warning if homoglyph normalization is implemented
- **Priority:** Low (but flag to product)

### TC-027 — Duplicate detection is updated after rename in DS-2
- **Preconditions:** Original "Web Development 2026" was renamed to "Web Development 2026 - Updated" via the edit feature (DS-2)
- **Steps:**
  1. Try to create a new program with Name = "Web Development 2026"
- **Expected result:** Creation succeeds — the old name is no longer in use (uniqueness is on the current value, not history)
- **Priority:** Medium

### TC-028 — Duplicate check is re-validated after first failure
- **Preconditions:** "Web Development 2026" exists
- **Steps:**
  1. Enter "Web Development 2026", click Create → see duplicate error
  2. Without closing the form, change Name to "Web Development 2026 v2"
  3. Click Create
- **Expected result:** Second submit succeeds (error is cleared once the field changes)
- **Priority:** Medium

### TC-029 — Pasted name with surrounding whitespace is normalized on display
- **Preconditions:** Logged in as admin
- **Steps:**
  1. Paste `"  Cloud Engineering 2026  "` into Name
  2. Observe field display (before submitting)
- **Expected result:** Either the field auto-trims on blur or shows a hint; on Submit, the stored value is trimmed (consistent with TC-004)
- **Priority:** Low

### TC-030 — Submitting via Enter key runs the same validation
- **Preconditions:** Logged in as admin, on the form
- **Steps:**
  1. Enter a whitespace-only or duplicate name
  2. Press Enter inside the Name field
- **Expected result:** Same validation as clicking Create; no submission; same error messages
- **Priority:** Low

### TC-031 — Long name boundary on UI does not visually break the list
- **Preconditions:** A program with maximum-length name exists
- **Steps:**
  1. Navigate to the Programs list
- **Expected result:** Layout stays intact; long name truncates with ellipsis + tooltip showing full value; no horizontal scroll on a typical viewport
- **Priority:** Low

### TC-032 — Duplicate check is debounced / not spam-firing during typing
- **Preconditions:** If real-time duplicate check exists (on blur or on keystroke)
- **Steps:**
  1. Type a name slowly into the Name field while watching the network panel
- **Expected result:** Duplicate-check requests are debounced (not one per keystroke); cancelled when value changes; final request reflects the actual submitted value
- **Priority:** Low

---

## Ambiguities & Gaps in the ACs

1. **Case sensitivity of uniqueness is undefined.** "Web Development 2026" vs "web development 2026" — duplicate or not? This drives TC-007 vs TC-013.
2. **Whitespace normalization scope.** ACs only say whitespace-only names are trimmed/rejected. Unclear if leading/trailing whitespace around real content is trimmed, and whether internal multiple spaces are collapsed (TC-004, TC-020).
3. **Length boundaries are not specified.** Min and max length for Name aren't defined (TC-005, TC-006, TC-014, TC-015).
4. **Allowed character set is not specified.** ACs accept `&` and `-`, but `<`, `>`, `'`, `"`, `;` are unaddressed. Important for security tests (TC-021, TC-022).
5. **Unicode/emoji support not specified.** Database column type and validation rules for 4-byte UTF-8, RTL, combining marks (TC-023, TC-024).
6. **Homoglyph / zero-width handling.** Not addressed (TC-026) — relevant if "data integrity" implies preventing visually identical names.
7. **Server-side enforcement.** ACs describe the form behavior, not the API contract. There must be a unique DB constraint + server validation so client-side checks can't be bypassed (TC-016, TC-018).
8. **Concurrency / race conditions.** Two admins creating the same name simultaneously is not addressed (TC-018).
9. **Error messaging.** Exact wording of duplicate / empty / length errors isn't specified.
10. **Soft-deleted programs.** If programs can be archived/deleted, does the name become available again? Affects TC-027.
11. **Scope of uniqueness.** Globally unique, unique per tenant/organization, or unique per year/category? Not stated.
12. **Validation timing.** On Submit only, on blur, or live-as-you-type? Affects TC-032 and UX expectations.
13. **Audit logging.** Should failed create attempts (duplicate / invalid) be logged for audit?

Recommend confirming items 1, 3, 4, 7, and 11 with PM before sign-off — they materially change the expected results above.
