# Spec: Registration

## Overview
Implement user registration so new visitors can create a Spendly account.
This step wires up the existing `POST /register` form to the database,
validates input server-side, hashes the password, inserts the new user row,
and redirects to the login page on success. It also sets a Flask `secret_key`
(required for flash messages and future session work) and introduces
`flash()` for user-facing feedback.

## Depends on
- Step 01 — Database Setup (`users` table must exist)

## Routes
- `GET  /register` — render registration form — public (already exists, no change)
- `POST /register` — validate form data, create user, redirect — public

## Database changes
No new tables or columns. The existing `users` table (from Step 01) is sufficient:
- `name TEXT NOT NULL`
- `email TEXT UNIQUE NOT NULL`
- `password_hash TEXT NOT NULL`

## Templates
- **Modify:** `templates/register.html`
  - Render `{{ get_flashed_messages() }}` for success/error feedback
  - Re-populate `name` and `email` fields on validation failure using `{{ request.form.name }}` / `{{ request.form.email }}`
  - Replace the static `{% if error %}` block with flashed messages

## Files to change
- `app.py`
  - Add `from flask import Flask, render_template, request, redirect, url_for, flash`
  - Set `app.secret_key` (read from env var `SECRET_KEY`, fall back to a dev default)
  - Change `@app.route("/register")` to accept `methods=["GET", "POST"]`
  - Implement `register()` view: validate → insert → flash → redirect

- `database/db.py`
  - Add `create_user(name, email, password)` — hashes password, inserts row, returns new `user_id`
  - Add `get_user_by_email(email)` — returns a `sqlite3.Row` or `None`

- `templates/register.html`
  - Swap `{% if error %}` error block for `{% with messages = get_flashed_messages(with_categories=true) %}` pattern
  - Add `value="{{ request.form.name }}"` and `value="{{ request.form.email }}"` to inputs

## Files to create
None.

## New dependencies
No new pip packages. Uses:
- `werkzeug.security.generate_password_hash` (already installed)
- `flask.flash`, `flask.request`, `flask.redirect`, `flask.url_for` (built into Flask)

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never string-format SQL
- Passwords hashed with `werkzeug.security.generate_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `secret_key` must never be hardcoded as a bare string in committed code — read from `os.environ.get("SECRET_KEY", "dev-secret-change-in-prod")`
- Duplicate email must be caught (UNIQUE constraint → `sqlite3.IntegrityError`) and shown as a user-friendly flash message, not a 500 error
- Server-side validation must check: name not blank, email not blank, password ≥ 8 characters — flash an error and re-render on failure
- On success: flash a success message, then `redirect(url_for("login"))`

## Definition of done
- [ ] `GET /register` renders the form (unchanged behaviour)
- [ ] Submitting the form with valid data inserts a new row in `users` with a hashed password
- [ ] After successful registration the user is redirected to `/login`
- [ ] A flash message "Account created — please sign in." is visible on the login page after redirect
- [ ] Submitting with a blank name, blank email, or password shorter than 8 characters shows a validation error and re-displays the form with name/email pre-filled
- [ ] Registering with an already-used email shows "An account with that email already exists." and does not insert a duplicate row
- [ ] Passwords are never stored in plain text (verify via `sqlite3` CLI or DB browser)
- [ ] App starts without errors when `SECRET_KEY` env var is not set (uses dev default)
