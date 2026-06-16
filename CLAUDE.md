# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`django-modern-user` is a Django package providing a custom user model (`ModernUser`) that replaces the default username-based authentication with case-insensitive email authentication, and removes first/last name fields. It is intended to be subclassed by downstream projects, not used directly as `AUTH_USER_MODEL`.

## Commands

```bash
# Install dependencies (venv is created in-project at .venv/)
poetry install

# Run tests
python manage.py test

# Run a specific test
python manage.py test django_modern_user.tests.ModernUserTests.test_lower_case_email

# Lint
flake8

# Format
black .

# Apply migrations (for local dev/test setup)
python manage.py migrate
```

## Architecture

The package lives in [django_modern_user/](django_modern_user/) and has three meaningful files:

- [models.py](django_modern_user/models.py) — `ModernUser` (subclasses `AbstractUser`) and `ModernUserManager` (subclasses `BaseUserManager`). Case-insensitivity is enforced in two places: `save()` lowercases the email before writing to DB, and `get_by_natural_key()` lowercases the lookup key for authentication.
- [admin.py](django_modern_user/admin.py) — `ModernUserAdmin` (subclasses `UserAdmin`) configured for email-only fieldsets. Consumers subclass this in their own `admin.py`.
- [tests.py](django_modern_user/tests.py) — all tests live here, run via Django's test runner.

The [core/](core/) directory is a minimal Django project used solely for local development and running tests. `AUTH_USER_MODEL` is set to `"django_modern_user.ModernUser"` in [core/settings.py](core/settings.py).

## Release Process

1. Commit code changes.
2. `poetry version <patch|minor|major>` to bump version in `pyproject.toml`.
3. Commit with the version as the message (e.g., `v5.0.0`).
4. Tag the commit: `git tag v5.0.0`.
5. Push commit and tag: `git push && git push --tags`.

CI/CD triggers on version tags (`v*.*.*`) and publishes to PyPI after tests pass across the full matrix (Python 3.10–3.13, Django 4.2/5.1/5.2).
