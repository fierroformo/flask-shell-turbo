# Flask Shell Turbo
A drop-in utility that supercharges the Flask interactive shell, inspired by Django's `shell_plus` (from `django-extensions`). It auto-discovers your SQLAlchemy models, injects common utilities, and gives you a productive REPL (Read-Eval-Print Loop) for debugging, data exploration, and quick fixes — no repetitive imports required.

## Description
Working with `flask shell` out of the box means manually importing every model and helper you need each time you open a session. **Flask Shell Turbo** solves this by:

- **Auto-discovering all SQLAlchemy models** registered on `db.Model`, so every model in your project is available in the shell immediately — no imports.
- **Injecting common utilities** like `datetime`, `date`, `timedelta`, `timezone`, and core SQLAlchemy query helpers (`func`, `select`, `text`, `and_`, `or_`, `not_`, `desc`, `asc`).
- **Using IPython automatically** when available, with `%autoreload` enabled so code changes are picked up without restarting the shell.
- **Providing safe helpers** — `commit()` and `rollback()` — that wrap `db.session` operations with clear success/error feedback.
- **Enhancing the standard `flask shell`** too, via a `shell_context_processor`, so you get the benefits even without the new command.
- **Adding a new CLI command**, `flask shell-turbo`, with flags for a plain Python fallback and SQL query logging.

## Requirements
- Flask
- Flask-SQLAlchemy (or a project using the `db.Model` declarative pattern)
- SQLAlchemy 1.4+ (a legacy fallback is included for older versions)
- IPython *(optional, but recommended)*

## Installation
1. Install flask-shell-turbo

   ```bash
   pip install flask-shell-turbo
   ```

## Integration into a Flask Project
Call `init_shell_turbo(app, db)` right after your `db` is initialized — typically in your app factory, after `db.init_app(app)`.

```python
from flask import Flask
from your_project.extensions import db
from flask_shell_turbo import init_shell_turbo

def create_app():
    app = Flask(__name__)
    # ... your config ...
    db.init_app(app)
    # Register shell_turbo after db is initialized
    init_shell_turbo(app, db)

    return app
```

That's it — no further configuration is needed. Model discovery happens automatically each time the shell starts, so newly added models are picked up without any manual registration.

## Usage

### Launch the enhanced shell
```bash
FLASK_APP=app flask shell-turbo
```

This drops you into an IPython session (or the standard Python shell if IPython isn't installed) with everything pre-loaded:

```
Flask shell-turbo
Models: Account, Order, User
Extras : db, session, func, select, and_, or_, desc, asc, datetime, date, timedelta, text
Helpers: commit(), rollback()
──────────────────────────────
```

### Work with your models directly — no imports
```python
user = User.query.filter_by(username="ab").first()
user.name = "Alejandro"
commit()
```

If something goes wrong, `commit()` automatically rolls back and prints the error instead of leaving your session in a broken transaction state:

```
✘ rollback — (psycopg2.errors.UniqueViolation) duplicate key value violates unique constraint...
```

### CLI flags
| Flag           | Description                                                              |
|----------------|---------------------------------------------------------------------------|
| `--plain`      | Forces the standard Python shell (`code.interact`) even if IPython is installed. |
| `--print-sql`  | Enables SQL echo (`db.engine.echo = True`) so every generated query is printed to the console. |

Example:
```bash
flask shell-turbo --print-sql
```

### Standard `flask shell` also benefits
Even without using `shell-turbo`, the regular command is enriched automatically:

```bash
flask shell
```

Because `init_shell_turbo` registers a `shell_context_processor`, all discovered models and utilities are available there too — `shell-turbo` simply adds the nicer IPython experience, autoreload, and the extra CLI flags on top.

## Benefits
- **Zero-import debugging** — jump straight into querying and modifying data without hunting down import paths.
- **Faster iteration** — `%autoreload` picks up code changes to your models and helpers without restarting the shell.
- **Safer transactions** — `commit()`/`rollback()` prevent half-finished transactions from silently blocking further queries.
- **Query visibility on demand** — `--print-sql` lets you inspect the exact SQL Flask-SQLAlchemy generates, useful for debugging N+1 queries or unexpected joins.
- **Works everywhere** — enhances both the new `shell-turbo` command and the existing `flask shell`, so teammates who haven't adopted it yet still benefit.
- **No dependencies required** — IPython is optional; the tool gracefully falls back to a standard shell with tab-completion via `readline`/`rlcompleter`.
- **Version-safe model discovery** — supports both the modern SQLAlchemy `registry.mappers` API and the legacy `_decl_class_registry`, so it works across SQLAlchemy versions.
