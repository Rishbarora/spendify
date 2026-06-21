# Spec: Login and Logout

## Overview
Implement session-based login and logout so registered users can authenticate
and maintain a signed-in state across requests. This step wires up the existing
`POST /login` form to the database, verifies the password hash, stores the user
identity in Flask's signed cookie session, and implements logout by clearing that
session. It also makes the navbar in `base.html` session-aware — showing
"Sign in / Get started" to guests and a greeting with a logout link to signed-in
users.

## Depends on
- Step 01 — Database Setup (`users` table and `get_user_by_email()` must exist)
- Step 02 — Registration (users must exist to log in; `secret_key` already set)

## Routes
- `GET  /login`  — render login form — public (already exists, no change)
- `POST /login`  — validate credentials, set session, redirect — public
- `GET  /logout` — clear session, redirect to landing — logged-in only

## Database changes
No new tables or columns. `get_user_by_email()` from Step 02 is sufficient.
Add `check_password_hash` import to `database/db.py` for a new helper.

## Templates
- **Modify:** `templates/login.html`
  - Add `methods=["GET", "POST"]` handling (template already has the form; no HTML change needed beyond what Step 02 added for flash messages)
- **Modify:** `templates/base.html`
  - Nav links must be conditional on `session.get("user_id")`:
    - Guest: show "Sign in" and "Get started" (current behaviour)
    - Logged in: show "Hello, {{ session.user_name }}" (non-clickable) and a "Sign out" link to `/logout`

## Files to change
- `app.py`
  - Add `session` to the Flask import
  - Change `@app.route("/login")` to accept `methods=["GET", "POST"]`
  - Implement `login()` view: validate → verify password → set session → redirect
  - Replace placeholder `logout()` view: clear session → flash → redirect to landing

- `database/db.py`
  - Add `check_password_hash` to the `werkzeug.security` import
  - Add `verify_user(email, password)` — looks up user by email, checks hash, returns the `sqlite3.Row` on success or `None` on failure

- `templates/base.html`
  - Replace static nav links with a Jinja2 `{% if session.user_id %}` conditional block

## Files to create
None.

## New dependencies
No new pip packages. Uses:
- `werkzeug.security.check_password_hash` (already installed)
- `flask.session` (built into Flask)

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never string-format SQL
- Passwords verified with `werkzeug.security.check_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Session must store exactly two keys: `user_id` (int) and `user_name` (str)
- `session.clear()` must be used in logout — do not manually delete individual keys
- After successful login, redirect to `url_for("profile")` (placeholder is fine for now)
- After logout, redirect to `url_for("landing")`
- `verify_user()` must never expose whether the email or the password was wrong — the view shows a single generic message: "Invalid email or password."
- Logout route must redirect even if the user is not logged in (no error)

## Definition of done
- [ ] `GET /login` renders the form (unchanged behaviour)
- [ ] Submitting valid credentials sets a session and redirects to `/profile`
- [ ] After login, the navbar shows "Hello, \<name\>" and a "Sign out" link instead of "Sign in / Get started"
- [ ] Submitting an unknown email shows "Invalid email or password." and re-renders the form with email pre-filled
- [ ] Submitting a wrong password shows the same generic error
- [ ] Visiting `/logout` clears the session and redirects to the landing page
- [ ] After logout, the navbar reverts to "Sign in / Get started"
- [ ] Visiting `/logout` when already signed out redirects cleanly without an error
- [ ] Refreshing any page after login still shows the logged-in nav (session persists across requests)
