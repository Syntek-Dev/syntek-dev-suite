# Stack: Django & Wagtail (Docker)

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Architecture](#architecture)
- [Commands](#commands)
- [Coding Standards](#coding-standards)
  - [Django](#django)
  - [Wagtail](#wagtail)
  - [Templates](#templates)
  - [Type Hinting](#type-hinting)
- [File Structure](#file-structure)
- [Testing (pytest)](#testing-pytest)

---

## Architecture

| Layer        | Technology                            |
| ------------ | ------------------------------------- |
| **Platform** | Raw Docker (Docker Compose)           |
| **Backend**  | Python 3.10+, Django 5.x, Wagtail 6.x |
| **Server**   | Gunicorn / Nginx                      |
| **Testing**  | pytest, pytest-django                 |

---

## Commands

| Task              | Command                                                            |
| ----------------- | ------------------------------------------------------------------ |
| Start environment | `docker compose up -d`                                             |
| Stop environment  | `docker compose down`                                              |
| View logs         | `docker compose logs -f web`                                       |
| Django manage     | `docker compose exec web python manage.py <command>`               |
| Django shell      | `docker compose exec web python manage.py shell`                   |
| Run tests         | `docker compose exec web pytest`                                   |
| Run specific test | `docker compose exec web pytest tests/test_file.py -k test_name`   |
| Migrations        | `docker compose exec web python manage.py migrate`                 |
| Make migrations   | `docker compose exec web python manage.py makemigrations`          |
| Create superuser  | `docker compose exec web python manage.py createsuperuser`         |
| Collect static    | `docker compose exec web python manage.py collectstatic --noinput` |
| Pip install       | `docker compose exec web pip install <package>`                    |

---

## Coding Standards

### Django
- Logic lives in **Services** or **Models**, not Views
- Views should be thin - delegate to services
- Use class-based views for CRUD operations
- Use function-based views for custom logic

### Wagtail
- Use `StreamField` for flexible content
- Split blocks into `blocks.py` files
- Use `StructBlock` for grouped fields
- Keep page models focused

### Templates
- Use DTL inheritance: `{% extends 'base.html' %}`
- Use `{% include %}` sparingly - prefer template inheritance
- Use `{% block %}` for overridable sections
- Keep logic minimal in templates

### Type Hinting
**CRITICAL:** All Python code must use strict type hints.

```python
from typing import Optional, List
from django.db.models import QuerySet

def get_active_users(limit: Optional[int] = None) -> QuerySet['User']:
    """
    Retrieves active users from the database.

    Args:
        limit: Maximum number of users to return. None returns all.

    Returns:
        QuerySet[User]: Active user records.
    """
    users = User.objects.filter(is_active=True)
    if limit:
        users = users[:limit]
    return users
```

---

## File Structure

```
project/
├── apps/
│   ├── core/               # Shared functionality
│   │   ├── models.py
│   │   ├── services.py     # Business logic
│   │   └── utils.py
│   ├── users/              # User management
│   └── content/            # Wagtail content
│       ├── models.py       # Page models
│       └── blocks.py       # StreamField blocks
├── templates/
│   ├── base.html
│   └── pages/
├── static/
├── tests/
│   ├── conftest.py         # pytest fixtures
│   ├── test_models.py
│   └── test_services.py
├── docs/
│   └── METRICS/            # Self-learning system (see global-workflow skill)
│       ├── README.md
│       ├── config.json
│       ├── runs/
│       ├── feedback/
│       └── optimisations/
├── manage.py
├── docker-compose.yml
└── requirements.txt
```

---

## Testing (pytest)

```python
import pytest
from apps.users.services import UserService

@pytest.fixture
def user_service() -> UserService:
    return UserService()

def test_create_user(user_service: UserService, db) -> None:
    """Test user creation with valid data."""
    user = user_service.create(
        email='test@example.com',
        name='Test User'
    )

    assert user.email == 'test@example.com'
    assert user.is_active is True
```
