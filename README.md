# Assessment Test: Software Tester (QA)
**Application Under Test:** Task Management Application
**Scope:** Registration, Login, Task CRUD, Input Validation & Error Handling
**Assumption:** Application is close to production release; no existing test documentation.

---

## 1. Test Case Design

### 1.1 Registration

**Test Scenarios**
- Verify a new user can register with valid details.
- Verify the system rejects invalid, incomplete, or duplicate registration data.
- Verify edge cases around field length, special characters, and concurrent registration.

| ID | Title | Type | Steps | Expected Result |
|---|---|---|---|---|
| REG-01 | Register with valid unique email, valid password, all required fields | Positive | 1. Open registration form 2. Enter valid name/email/password 3. Submit | Account is created; confirmation shown/email sent; user redirected to login or dashboard |
| REG-02 | Register with an email already in use | Negative | Enter an email that already exists, submit | Registration blocked; clear "email already registered" error; no duplicate account created |
| REG-03 | Register with invalid email format (e.g. `abc@`, `abc.com`, `a@@b.com`) | Negative | Enter malformed email, submit | Inline validation error; form not submitted |
| REG-04 | Register with missing required field (name/email/password blank) | Negative | Leave one required field empty, submit | Field-specific error shown; submission blocked |
| REG-05 | Register with weak password (below minimum length/complexity) | Negative | Enter e.g. "123", submit | Password policy error shown; account not created |
| REG-06 | Register with password/confirm-password mismatch (if applicable) | Negative | Enter differing values | Mismatch error shown |
| REG-07 | Register with max-length boundary values for name/email | Edge | Enter name/email at the documented max character limit | Accepted if within limit; rejected gracefully if exceeded (not truncated silently) |
| REG-08 | Register with leading/trailing spaces in email | Edge | " user@test.com " | Either trimmed automatically and accepted, or rejected consistently — should not create a malformed record |
| REG-09 | Register with special characters/emoji/SQL-like input in name field | Edge/Security | Enter `Robert'); DROP TABLE users;--` or emoji | Input sanitized/escaped; no script/SQL execution; no crash |
| REG-10 | Submit registration form twice in quick succession (double-click submit) | Edge | Rapidly double-click submit | Only one account created; no duplicate records or duplicate emails sent |

### 1.2 Login

**Test Scenarios**
- Verify a registered user can log in with correct credentials.
- Verify incorrect/invalid credentials are rejected with appropriate feedback.
- Verify session, lockout, and edge-case handling.

| ID | Title | Type | Steps | Expected Result |
|---|---|---|---|---|
| LOG-01 | Login with valid registered email + correct password | Positive | Enter valid credentials, submit | User authenticated; redirected to dashboard/task list |
| LOG-02 | Login with correct email, wrong password | Negative | Enter valid email, incorrect password | Generic "invalid credentials" error (should not reveal which field was wrong) |
| LOG-03 | Login with unregistered email | Negative | Enter email not in system | Generic invalid-credentials error (not "email not found," to avoid user enumeration) |
| LOG-04 | Login with empty email/password fields | Negative | Submit blank form | Validation error; no request sent to server unnecessarily |
| LOG-05 | Repeated failed login attempts (brute-force check) | Negative/Security | Attempt wrong password 5–10 times | Account lockout / CAPTCHA / rate-limiting triggers after threshold |
| LOG-06 | Login with case-sensitivity variation in email (`User@Test.com` vs `user@test.com`) | Edge | Register with lowercase, log in with mixed case | Should succeed (emails are typically case-insensitive) — flag if it fails |
| LOG-07 | Session persistence / "Remember me" | Edge | Log in, close browser, reopen | Session behaves as expected per "remember me" setting |
| LOG-08 | Session expiry / token timeout | Edge | Stay idle beyond session timeout, then perform an action | User is prompted to re-authenticate; no silent failure or data loss |
| LOG-09 | Login via direct URL manipulation while unauthenticated | Security | Navigate directly to `/dashboard` without logging in | Redirected to login page; no data exposed |
| LOG-10 | SQL injection / script injection in login fields | Security | Enter `' OR '1'='1` in email/password | Login rejected safely; no bypass, no error leaking stack trace |

### 1.3 Task CRUD Operations

**Test Scenarios**
- Verify authenticated users can create, view, edit, and delete tasks.
- Verify tasks persist correctly and are scoped to the correct user.
- Verify boundary and concurrency behavior.

| ID | Title | Type | Steps | Expected Result |
|---|---|---|---|---|
| TASK-01 | Create a task with valid required fields (title, etc.) | Positive | Fill task form, save | Task appears in task list immediately, persists after refresh |
| TASK-02 | View task list after login | Positive | Log in, navigate to task list | All of the logged-in user's tasks display correctly; no other user's tasks visible |
| TASK-03 | Edit an existing task's details | Positive | Open task, change title/description/due date, save | Updated values persist and display correctly in list and detail view |
| TASK-04 | Delete a task | Positive | Select task, delete, confirm | Task removed from list and database; not recoverable unless "undo"/soft-delete is a feature |
| TASK-05 | Create a task with empty/blank required field (e.g., title) | Negative | Leave title blank, submit | Validation error; task not created |
| TASK-06 | Edit a task that no longer exists (deleted by another session/tab) | Negative/Edge | Open task in Tab A, delete it in Tab B, then save edit in Tab A | Graceful error ("task no longer exists"), not a crash or silent overwrite |
| TASK-07 | Create task with very long title/description (boundary length) | Edge | Enter text at/above max character limit | Enforced limit with clear error, or safely truncated — not a DB error or UI overflow |
| TASK-08 | Create task with special characters/HTML/script tags in title | Security/Edge | Enter `<script>alert(1)</script>` | Rendered as plain text (escaped), not executed — checks for stored XSS |
| TASK-09 | Delete confirmation cancel flow | Negative/Edge | Click delete, then cancel the confirmation | Task remains unchanged in the list |
| TASK-10 | Pagination/large list performance | Edge | User with 500+ tasks views task list | List loads within acceptable time; pagination/infinite scroll works; no missing/duplicated entries |
| TASK-11 | Concurrent edits by the same user in two tabs | Edge | Edit same task simultaneously in two tabs, save both | Last-write-wins behavior is consistent and doesn't corrupt data; ideally a conflict warning |
| TASK-12 | Task ownership / access control (IDOR check) | Security | User A tries to access/edit User B's task via direct task ID/URL manipulation | Access denied; user cannot view or modify another user's task |

### 1.4 Input Validation & Error Handling (Cross-Cutting)

| ID | Title | Type | Steps | Expected Result |
|---|---|---|---|---|
| VAL-01 | All forms handle unexpected server errors (500) gracefully | Negative | Simulate/observe server error during submit | User-friendly error message shown; no raw stack trace or blank screen |
| VAL-02 | Network failure during save/delete | Edge | Disable network mid-request | Clear "failed to save, please retry" message; no silent data loss; no duplicate submission on retry |
| VAL-03 | Date fields accept only valid dates (e.g., task due date) | Negative | Enter invalid date like 31/02/2026 or a past date if not allowed | Validation error shown; invalid dates rejected |
| VAL-04 | File/field size limits enforced consistently front-end and back-end | Negative/Security | Bypass front-end validation (e.g., via browser dev tools or direct API call) and submit invalid data | Back-end validation still rejects it — front-end validation alone is not trusted |
| VAL-05 | Consistent error messaging/formatting across the app | Edge | Trigger validation errors on each form | Errors are consistent in tone, placement, and clarity across Registration/Login/Task forms |

---

## 2. Bug Identification — Potential Bugs / Risk Areas
*(Identified through static/requirement analysis, without executing the application)*

| # | Bug / Risk Area | Severity | Reason / Impact |
|---|---|---|---|
| 1 | Front-end-only validation with no server-side re-validation | **Critical** | An attacker or a modified client request could submit invalid/malicious data directly to the API, corrupting data or enabling injection attacks. |
| 2 | User enumeration via differing error messages ("email not found" vs "wrong password") | **Major** | Reveals which emails are registered, aiding credential-stuffing/phishing attacks. |
| 3 | Missing authorization checks on task endpoints (IDOR) | **Critical** | A user could view, edit, or delete another user's tasks by guessing/changing a task ID in the URL or API request — a serious data privacy/security issue. |
| 4 | No rate-limiting or account lockout on login | **Major** | Enables brute-force password attacks against user accounts. |
| 5 | Lack of input sanitization allowing stored XSS in task title/description | **Critical** | Malicious script could execute in other sessions/admin views when the task is rendered, leading to session hijacking or data theft. |
| 6 | No handling for duplicate/double form submission (e.g., double-clicking "Create Task" or "Register") | **Minor** | Could create duplicate tasks or duplicate accounts, causing data inconsistency and user confusion. |
| 7 | Session timeout not clearly communicated to the user | **Minor** | User may lose unsaved task edits without warning if session expires mid-edit, causing frustration and possible data loss. |
| 8 | No confirmation/undo on task deletion described as final | **Major** | Accidental deletions are unrecoverable, which is a significant usability and data-loss risk for a production task manager. |
| 9 | Pagination/performance not specified for large task lists | **Minor** | Users with many tasks could experience slow load times or a broken/inconsistent list if pagination isn't implemented or tested at scale. |
| 10 | Case-sensitivity inconsistency in email handling between registration and login | **Minor** | If registration stores email as-is but login compares case-sensitively (or vice versa), legitimate users could be locked out of their own accounts. |

---

## 3. Notes on Approach

- Test design followed the **positive / negative / edge** structure requested, layering **security-relevant edge cases** (injection, IDOR, XSS) on top of standard functional cases, since the brief states the app is close to production.
- Bug identification was done through **static analysis of the described feature set** — inferring likely risk areas common to CRUD + auth applications (authorization, input handling, concurrency, and session management) rather than guessing implementation-specific bugs.
- Given the 1:15 hr time-box, deeper areas (e.g., full API contract testing, load testing, accessibility testing) were intentionally scoped out but would be natural next steps in a real test plan.
