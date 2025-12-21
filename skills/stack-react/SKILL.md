# Stack: React Web (Docker)

This skill defines the React web stack configuration. Apply when working on React web projects.

---

## Table of Contents

- [Architecture](#architecture)
- [Commands](#commands)
- [Coding Standards](#coding-standards)
- [File Structure](#file-structure)
- [Testing (Vitest + RTL)](#testing-vitest--rtl)

---

## Architecture

| Layer | Technology |
|-------|------------|
| **Platform** | Raw Docker (Node environment) |
| **Framework** | React 18+, TypeScript |
| **Build** | Vite |
| **Styling** | Tailwind CSS |
| **Testing** | Vitest, React Testing Library |

---

## Commands

| Task | Command |
|------|---------|
| Start environment | `docker compose up` |
| Start (detached) | `docker compose up -d` |
| Stop environment | `docker compose down` |
| Install packages | `docker compose run --rm app npm install` |
| Add package | `docker compose run --rm app npm install <package>` |
| Run tests | `docker compose run --rm app npm test` |
| Run tests (watch) | `docker compose run --rm app npm test -- --watch` |
| Build | `docker compose run --rm app npm run build` |
| Lint | `docker compose run --rm app npm run lint` |
| Shell | `docker compose run --rm app /bin/sh` |

---

## Coding Standards

### Components
- **Functional components only** - no class components
- **One component per file** with named exports
- **Strict TypeScript interfaces** for all props
- Use `React.FC` sparingly - prefer explicit return types

```tsx
interface UserCardProps {
  user: User;
  onSelect: (userId: string) => void;
  showBadge?: boolean;
}

export function UserCard({ user, onSelect, showBadge = true }: UserCardProps): JSX.Element {
  return (
    <div className="rounded-lg bg-white p-4 shadow">
      <h3 className="text-lg font-semibold">{user.name}</h3>
      {showBadge && <Badge status={user.status} />}
      <button onClick={() => onSelect(user.id)}>Select</button>
    </div>
  );
}
```

### State Management
| Size | Solution |
|------|----------|
| Component-local | `useState`, `useReducer` |
| Shared (small app) | Context API |
| Shared (medium app) | Zustand |
| Complex (large app) | Redux Toolkit (only if truly needed) |

### Styling (Tailwind)
- **Utility-first** approach
- Extract repeated patterns to `@layer components`
- Use design tokens from config
- Check for Tailwind v3 vs v4 configuration

### Hooks
- Prefix with `use` (e.g., `useAuth`, `useFetch`)
- Keep hooks focused on single responsibility
- Extract complex logic to custom hooks

---

## File Structure

```
src/
├── components/
│   ├── ui/                 # Generic UI components
│   │   ├── Button.tsx
│   │   └── Modal.tsx
│   └── features/           # Feature-specific components
│       └── UserCard.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useFetch.ts
├── contexts/
│   └── AuthContext.tsx
├── services/
│   └── api.ts              # API client
├── types/
│   └── index.ts            # Shared TypeScript types
├── utils/
│   └── formatters.ts
├── pages/                  # Route pages (if using router)
├── App.tsx
└── main.tsx

docs/
└── METRICS/            # Self-learning system (see global-workflow skill)
    ├── README.md
    ├── config.json
    ├── runs/
    ├── feedback/
    └── optimisations/
```

---

## Testing (Vitest + RTL)

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = { id: '1', name: 'Test User', status: 'active' };

  it('renders user name', () => {
    render(<UserCard user={mockUser} onSelect={vi.fn()} />);

    expect(screen.getByText('Test User')).toBeInTheDocument();
  });

  it('calls onSelect when clicked', () => {
    const onSelect = vi.fn();
    render(<UserCard user={mockUser} onSelect={onSelect} />);

    fireEvent.click(screen.getByRole('button'));

    expect(onSelect).toHaveBeenCalledWith('1');
  });
});
```
