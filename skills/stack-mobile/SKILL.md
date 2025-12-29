# Stack: React Native Mobile

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
  - [Components](#components)
  - [Styling (NativeWind)](#styling-nativewind)
  - [Platform-Specific Code](#platform-specific-code)
  - [Navigation (React Navigation)](#navigation-react-navigation)
- [File Structure](#file-structure)
- [Platform Considerations](#platform-considerations)
  - [iOS Specific](#ios-specific)
  - [Android Specific](#android-specific)
- [Testing (Jest + RNTL)](#testing-jest--rntl)

---

## Architecture

| Layer          | Technology                                |
| -------------- | ----------------------------------------- |
| **Platform**   | Native / Simulator (NO Docker)            |
| **Framework**  | React Native, Expo (optional), TypeScript |
| **Styling**    | NativeWind (Tailwind for Mobile)          |
| **Navigation** | React Navigation                          |
| **Testing**    | Jest, React Native Testing Library        |

**CRITICAL:** Do NOT suggest Docker commands for this stack. Mobile runs natively.

---

## Commands

| Task                  | Command                                  |
| --------------------- | ---------------------------------------- |
| Start Metro Bundler   | `npm start`                              |
| Run on iOS            | `npm run ios`                            |
| Run on Android        | `npm run android`                        |
| Install iOS Pods      | `cd ios && pod install && cd ..`         |
| Run tests             | `npm test`                               |
| Run tests (watch)     | `npm test -- --watch`                    |
| Install packages      | `npm install <package>`                  |
| Lint                  | `npm run lint`                           |
| Clean build (iOS)     | `cd ios && rm -rf build && cd ..`        |
| Clean build (Android) | `cd android && ./gradlew clean && cd ..` |
| Reset Metro cache     | `npm start -- --reset-cache`             |

---

## Coding Standards

### Components
- **Use React Native Core Components:** `<View>`, `<Text>`, `<ScrollView>`, `<TouchableOpacity>`
- **NEVER use HTML elements:** No `<div>`, `<span>`, `<p>`, `<button>`
- **Functional components only** with TypeScript interfaces

```tsx
import { View, Text, TouchableOpacity } from 'react-native';

interface UserCardProps {
  user: User;
  onPress: (userId: string) => void;
}

export function UserCard({ user, onPress }: UserCardProps): JSX.Element {
  return (
    <TouchableOpacity
      className="rounded-lg bg-white p-4 shadow-md"
      onPress={() => onPress(user.id)}
    >
      <Text className="text-lg font-semibold text-gray-900">{user.name}</Text>
      <Text className="text-sm text-gray-500">{user.email}</Text>
    </TouchableOpacity>
  );
}
```

### Styling (NativeWind)
- Use Tailwind-like classes: `className="flex-1 bg-white"`
- Responsive design not pixel-based - use flex
- Test on both iOS and Android for styling differences

### Platform-Specific Code
| Pattern                                           | Use Case                 |
| ------------------------------------------------- | ------------------------ |
| `Platform.OS === 'ios'`                           | Inline conditionals      |
| `Component.ios.tsx` / `Component.android.tsx`     | Separate component files |
| `Platform.select({ ios: {...}, android: {...} })` | Style objects            |

### Navigation (React Navigation)
- Use typed navigation with TypeScript
- Define navigation types centrally
- Use Stack, Tab, and Drawer navigators appropriately

---

## File Structure

```
src/
├── components/
│   ├── ui/                   # Generic UI components
│   │   ├── Button.tsx
│   │   └── Card.tsx
│   └── features/             # Feature-specific components
├── screens/                  # Screen components
│   ├── HomeScreen.tsx
│   └── ProfileScreen.tsx
├── navigation/
│   ├── types.ts              # Navigation type definitions
│   ├── RootNavigator.tsx
│   └── TabNavigator.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useStorage.ts
├── services/
│   └── api.ts
├── types/
│   └── index.ts
├── utils/
│   └── formatters.ts
└── App.tsx

docs/
└── METRICS/            # Self-learning system (see global-workflow skill)
    ├── README.md
    ├── config.json
    ├── runs/
    ├── feedback/
    └── optimisations/
```

---

## Platform Considerations

### iOS Specific
- Handle safe area insets (`SafeAreaView`)
- Face ID / Touch ID authentication
- Test on notched devices (iPhone X+)
- Handle keyboard avoiding view

### Android Specific
- Handle back button navigation
- Status bar configuration
- Handle different screen densities
- Test on various Android versions

---

## Testing (Jest + RNTL)

```tsx
import { render, fireEvent, screen } from '@testing-library/react-native';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = { id: '1', name: 'Test User', email: 'test@example.com' };

  it('renders user information', () => {
    render(<UserCard user={mockUser} onPress={jest.fn()} />);

    expect(screen.getByText('Test User')).toBeOnTheScreen();
    expect(screen.getByText('test@example.com')).toBeOnTheScreen();
  });

  it('calls onPress when tapped', () => {
    const onPress = jest.fn();
    render(<UserCard user={mockUser} onPress={onPress} />);

    fireEvent.press(screen.getByText('Test User'));

    expect(onPress).toHaveBeenCalledWith('1');
  });
});
```
