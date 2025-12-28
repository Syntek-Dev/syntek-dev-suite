# Version History Template

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Template](#template)
- [Section Guidelines](#section-guidelines)
  - [Summary](#summary)
  - [Breaking Changes](#breaking-changes)
  - [Database Migrations](#database-migrations)
  - [API Changes](#api-changes)
  - [Files Changed](#files-changed)
  - [Dependencies Updated](#dependencies-updated)
  - [Configuration Changes](#configuration-changes)
  - [Performance Notes](#performance-notes)
  - [Security Notes](#security-notes)
- [Examples](#examples)
  - [Example: Feature Release (MINOR)](#example-feature-release-minor)
  - [Example: Bug Fix Release (PATCH)](#example-bug-fix-release-patch)
  - [Example: Breaking Change Release (MAJOR)](#example-breaking-change-release-major)
- [Best Practices](#best-practices)

---

## Overview

The `VERSION-HISTORY.md` file is a **technical changelog** intended for developers. It contains detailed information about:

- Specific code changes with file paths
- Database migrations and schema changes
- API endpoint modifications
- Breaking changes with migration paths
- Dependency updates
- Configuration changes
- Performance and security notes

This differs from `CHANGELOG.md` (brief summary) and `RELEASES.md` (user-facing).

---

## Template

Copy this template to your project root as `VERSION-HISTORY.md`:

```markdown
# Version History

**Last Updated**: DD/MM/YYYY
**Version**: X.Y.Z
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

All technical changes to this project are documented in this file.

This is a detailed technical changelog for developers. For a brief summary, see [CHANGELOG.md](CHANGELOG.md). For user-facing release notes, see [RELEASES.md](RELEASES.md).

---

## Table of Contents

- [Unreleased](#unreleased)

---

## [Unreleased]

### Summary
No unreleased changes.

### Breaking Changes
None.

### Database Migrations
None.

### API Changes
None.

### Files Changed
None.

### Dependencies Updated
None.

### Configuration Changes
None.

### Performance Notes
None.

### Security Notes
None.

---

## [1.0.0] - DD/MM/YYYY

### Summary
Initial release of the application.

### Breaking Changes
| Change | Migration Path | Affected Files |
| ------ | -------------- | -------------- |
| N/A    | N/A            | N/A            |

### Database Migrations
| Migration                              | Description             | Reversible |
| -------------------------------------- | ----------------------- | ---------- |
| `2025_01_01_000000_create_users_table` | Creates the users table | Yes        |

### API Changes
| Endpoint             | Method | Change                 |
| -------------------- | ------ | ---------------------- |
| `/api/auth/login`    | POST   | Initial implementation |
| `/api/auth/register` | POST   | Initial implementation |

### Files Changed
| File                                      | Changes                         |
| ----------------------------------------- | ------------------------------- |
| `app/Models/User.php`                     | Initial model creation          |
| `app/Http/Controllers/AuthController.php` | Login and registration handlers |

### Dependencies Updated
| Package | From | To  | Notes                |
| ------- | ---- | --- | -------------------- |
| N/A     | N/A  | N/A | Initial dependencies |

### Configuration Changes
| File           | Key | Change                         |
| -------------- | --- | ------------------------------ |
| `.env.example` | All | Initial configuration template |

### Performance Notes
- Baseline performance established

### Security Notes
- CSRF protection enabled
- Password hashing using bcrypt
```

---

## Section Guidelines

### Summary
A brief 1-2 sentence overview of what this version includes.

### Breaking Changes
**Required for MAJOR version bumps.**

| Column         | Description                   |
| -------------- | ----------------------------- |
| Change         | What specifically changed     |
| Migration Path | Steps to update existing code |
| Affected Files | Files that need updating      |

### Database Migrations
List all new migrations with:
- Migration filename
- Description of what it does
- Whether it can be reversed

### API Changes
Document changes to:
- New endpoints
- Modified endpoints (parameter changes, response changes)
- Deprecated endpoints
- Removed endpoints

### Files Changed
List significant file changes with:
- File path
- Description of changes

**Note:** For minor changes like typo fixes, you don't need to list every file.

### Dependencies Updated
Track package updates with:
- Package name
- Previous version
- New version
- Any notes about breaking changes

### Configuration Changes
Document new or changed config:
- Configuration file
- Key or setting name
- What changed

### Performance Notes
Document any:
- Performance improvements
- New caching implementations
- Query optimisations
- Load time changes

### Security Notes
Document any:
- Security patches
- CVE fixes
- New security features
- Authentication/authorisation changes

---

## Examples

### Example: Feature Release (MINOR)

```markdown
## [1.3.0] - 15/01/2025

### Summary
Added role-based access control and improved authentication flow.

### Breaking Changes
None.

### Database Migrations
| Migration                                | Description                          | Reversible |
| ---------------------------------------- | ------------------------------------ | ---------- |
| `2025_01_15_100000_create_roles_table`   | Creates roles and permissions tables | Yes        |
| `2025_01_15_100001_add_role_id_to_users` | Adds role_id foreign key to users    | Yes        |

### API Changes
| Endpoint               | Method | Change                                     |
| ---------------------- | ------ | ------------------------------------------ |
| `/api/users`           | GET    | Now requires `users.view` permission       |
| `/api/roles`           | GET    | New endpoint - lists available roles       |
| `/api/roles`           | POST   | New endpoint - creates a role (admin only) |
| `/api/users/{id}/role` | PUT    | New endpoint - assigns role to user        |

### Files Changed
| File                                      | Changes                              |
| ----------------------------------------- | ------------------------------------ |
| `app/Models/Role.php`                     | New model for roles                  |
| `app/Models/Permission.php`               | New model for permissions            |
| `app/Models/User.php`                     | Added `role()` relationship          |
| `app/Http/Middleware/CheckPermission.php` | New middleware for permission checks |
| `app/Http/Controllers/RoleController.php` | CRUD operations for roles            |
| `routes/api.php`                          | Added role management routes         |

### Dependencies Updated
| Package                     | From | To  | Notes                   |
| --------------------------- | ---- | --- | ----------------------- |
| `spatie/laravel-permission` | -    | 6.0 | New dependency for RBAC |

### Configuration Changes
| File                    | Key                 | Change                      |
| ----------------------- | ------------------- | --------------------------- |
| `config/permission.php` | `models.role`       | Points to custom Role model |
| `.env.example`          | `CACHE_PERMISSIONS` | New optional setting        |

### Performance Notes
- Permissions are cached for 24 hours by default
- Added database indexes on role_id columns

### Security Notes
- All API endpoints now require explicit permission grants
- Super-admin role bypasses permission checks
```

### Example: Bug Fix Release (PATCH)

```markdown
## [1.2.1] - 10/01/2025

### Summary
Fixed critical timezone bug and improved error handling.

### Breaking Changes
None.

### Database Migrations
None.

### API Changes
| Endpoint                 | Method | Change                                 |
| ------------------------ | ------ | -------------------------------------- |
| `/api/reports/scheduled` | GET    | Now returns UTC timestamps (was local) |

### Files Changed
| File                                 | Changes                                         |
| ------------------------------------ | ----------------------------------------------- |
| `app/Services/ReportScheduler.php`   | Fixed timezone conversion in `getNextRunTime()` |
| `app/Exceptions/Handler.php`         | Added logging for unhandled exceptions          |
| `tests/Unit/ReportSchedulerTest.php` | Added regression tests for timezone handling    |

### Dependencies Updated
None.

### Configuration Changes
| File           | Key            | Change                 |
| -------------- | -------------- | ---------------------- |
| `.env.example` | `APP_TIMEZONE` | Documented as required |

### Performance Notes
None.

### Security Notes
- Improved error messages no longer leak internal paths
```

### Example: Breaking Change Release (MAJOR)

```markdown
## [2.0.0] - 01/02/2025

### Summary
Major authentication overhaul: migrated from session-based to JWT authentication.

### Breaking Changes
| Change                                | Migration Path                                                                                                                               | Affected Files             |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| Sessions replaced with JWT            | 1. Update API calls to include `Authorization: Bearer <token>` header<br>2. Remove session cookie handling<br>3. Store JWT token client-side | All API consumers          |
| `/api/auth/login` response changed    | Update code to read `token` from response instead of relying on session                                                                      | `AuthService.ts`, `api.js` |
| `/api/auth/logout` now requires token | Include bearer token in logout request                                                                                                       | `AuthService.ts`           |
| `remember_me` parameter removed       | JWT tokens have configurable expiry via `JWT_TTL` env var                                                                                    | Login forms                |

### Database Migrations
| Migration                                    | Description                      | Reversible |
| -------------------------------------------- | -------------------------------- | ---------- |
| `2025_02_01_000000_drop_sessions_table`      | Removes the sessions table       | No         |
| `2025_02_01_000001_add_jwt_columns_to_users` | Adds token_invalidated_at column | Yes        |

### API Changes
| Endpoint            | Method | Change                                                     |
| ------------------- | ------ | ---------------------------------------------------------- |
| `/api/auth/login`   | POST   | Returns `{ token, expires_at }` instead of setting session |
| `/api/auth/logout`  | POST   | Requires bearer token, invalidates token                   |
| `/api/auth/refresh` | POST   | New endpoint - refreshes expiring token                    |
| `/api/auth/me`      | GET    | Now requires bearer token instead of session               |

### Files Changed
| File                                      | Changes                                     |
| ----------------------------------------- | ------------------------------------------- |
| `app/Http/Middleware/Authenticate.php`    | Switched from session to JWT validation     |
| `app/Http/Controllers/AuthController.php` | Complete rewrite for JWT flow               |
| `app/Services/JwtService.php`             | New service for token generation/validation |
| `config/auth.php`                         | Changed api guard to JWT driver             |
| `routes/api.php`                          | Added refresh token route                   |

### Dependencies Updated
| Package           | From | To  | Notes                           |
| ----------------- | ---- | --- | ------------------------------- |
| `tymon/jwt-auth`  | -    | 2.0 | New dependency for JWT handling |
| `laravel/sanctum` | 3.x  | -   | Removed, replaced by JWT        |

### Configuration Changes
| File             | Key               | Change                                                |
| ---------------- | ----------------- | ----------------------------------------------------- |
| `.env`           | `JWT_SECRET`      | **Required** - Generate with `php artisan jwt:secret` |
| `.env`           | `JWT_TTL`         | Token lifetime in minutes (default: 60)               |
| `.env`           | `JWT_REFRESH_TTL` | Refresh token lifetime (default: 20160 = 2 weeks)     |
| `config/jwt.php` | All               | New configuration file for JWT settings               |

### Performance Notes
- JWT validation is stateless, reducing database queries per request
- Token refresh prevents need for frequent re-authentication
- Removed session garbage collection overhead

### Security Notes
- Tokens are signed using RS256 algorithm
- Refresh tokens can be invalidated server-side
- Added rate limiting on `/api/auth/login` (5 attempts per minute)
- All tokens invalidated on password change
```

---

## Best Practices

1. **Update VERSION-HISTORY.md with every version bump**
2. **Be specific** - Include file paths, endpoint names, migration filenames
3. **Document migration paths** for breaking changes
4. **Include reversibility** for database migrations
5. **Note security implications** of any changes
6. **Use British English** spelling throughout
7. **Date format** - Always DD/MM/YYYY
