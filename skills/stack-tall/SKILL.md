# Stack: TALL (Laravel + DDEV)

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Architecture](#architecture)
- [Commands](#commands)
- [Coding Standards](#coding-standards)
  - [Livewire](#livewire)
  - [Blade Templates](#blade-templates)
  - [Alpine.js](#alpinejs)
  - [Testing (Pest)](#testing-pest)
- [File Structure](#file-structure)

---

## Architecture

| Layer        | Technology                   |
| ------------ | ---------------------------- |
| **Platform** | DDEV (Docker abstraction)    |
| **Backend**  | Laravel 11.x, Livewire 3     |
| **Frontend** | TailwindCSS, Alpine.js, Vite |
| **Testing**  | Pest PHP                     |

---

## Commands

**CRITICAL:** Never use raw `docker` commands. Always use DDEV wrappers.

| Task              | Command                                        |
| ----------------- | ---------------------------------------------- |
| Start environment | `ddev start`                                   |
| Stop environment  | `ddev stop`                                    |
| Artisan commands  | `ddev artisan <command>`                       |
| Composer          | `ddev composer <command>`                      |
| NPM               | `ddev exec npm <command>`                      |
| Run tests         | `ddev exec php artisan test`                   |
| Run specific test | `ddev exec php artisan test --filter=TestName` |
| Database import   | `ddev import-db < dump.sql`                    |
| Database export   | `ddev export-db > dump.sql`                    |
| Migrations        | `ddev artisan migrate`                         |
| Fresh migrate     | `ddev artisan migrate:fresh --seed`            |
| Tinker            | `ddev artisan tinker`                          |

---

## Coding Standards

### Livewire
- Use `wire:navigate` for SPA-like navigation
- Use `#[Rule]` attributes for validation
- Keep components focused (single responsibility)
- Use `#[Computed]` for derived data

### Blade Templates
- Use `<x-components>` exclusively
- **Never** use `@include` - extract to components instead
- Use slots for flexible component content
- Follow component naming: `<x-module.component-name>`

### Alpine.js
- Use `x-data` in separate JS files for complex logic
- Keep inline `x-data` for simple toggle states only
- Use `$wire` for Livewire integration
- Prefer `x-cloak` to prevent flash of unstyled content

### Testing (Pest)
```php
it('can create a user', function () {
    // Arrange
    $data = ['name' => 'Test User', 'email' => 'test@example.com'];

    // Act
    $response = post('/users', $data);

    // Assert
    $response->assertStatus(201);
    expect(User::count())->toBe(1);
});
```

---

## File Structure

```
app/
├── Livewire/           # Livewire components
├── Models/             # Eloquent models
├── Services/           # Business logic
├── Actions/            # Single-purpose actions
└── Http/
    ├── Controllers/    # Minimal, delegate to services
    └── Requests/       # Form validation

resources/
├── views/
│   ├── components/     # Blade components
│   ├── livewire/       # Livewire views
│   └── layouts/        # Layout templates
└── js/
    └── alpine/         # Complex Alpine components

tests/
├── Feature/            # Integration tests
└── Unit/               # Unit tests

docs/
└── METRICS/            # Self-learning system (see global-workflow skill)
    ├── README.md
    ├── config.json
    ├── runs/
    ├── feedback/
    └── optimisations/
```
