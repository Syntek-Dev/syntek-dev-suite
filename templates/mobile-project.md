# React Native Mobile App Template

Use this template to set up a React Native mobile app that consumes a GraphQL API from the Django backend and uses the shared library for styling/components.

**Template Repository:** `Syntek-Studio/mobile_template`

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Template Repository](#template-repository)
- [Project CLAUDE.md](#project-claudemd)
- [Directory Structure](#directory-structure)
- [Environment-Specific Command Files](#environment-specific-command-files)
- [GraphQL Code Generator Configuration](#graphql-code-generator-configuration)
- [Environment Variables](#environment-variables)
- [Package Dependencies](#package-dependencies)

---

## Architecture Overview

This is a **mobile app** that:
- **Consumes** GraphQL API from `stack-django` (Django/Wagtail backend)
- **Imports** components, typography, fonts, and colours from `stack-shared-lib`
- **Shares** the same design system with `stack-react` (React web)

```
┌─────────────────────────────────────────────────────────────────┐
│                        THIS PROJECT                              │
│                   React Native Mobile App                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  UI Layer                                                  │  │
│  │  - React Native with TypeScript                           │  │
│  │  - NativeWind (Tailwind for RN)                           │  │
│  │  - Components from @company/shared-lib                    │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Data Layer                                                │  │
│  │  - Apollo Client                                          │  │
│  │  - GraphQL queries and mutations                          │  │
│  │  - Generated TypeScript types from schema                 │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              │ GraphQL API
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Django Backend                               │
│                    (stack-django)                                │
│  - GraphQL API at /graphql/                                     │
│  - JWT Authentication                                           │
│  - PostgreSQL Database                                          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     Shared Library                               │
│                   (stack-shared-lib)                             │
│  - Components (Button, Card, Modal, etc.)                       │
│  - Typography system                                            │
│  - Colour palette and design tokens                             │
│  - Fonts                                                        │
└─────────────────────────────────────────────────────────────────┘
```

**CRITICAL:** This project runs **natively** on iOS/Android. **Do NOT use Docker commands.**

---

## Template Repository

**GitHub:** `Syntek-Studio/mobile_template`

### Creating a New Project from Template

```bash
# Option 1: GitHub template (recommended)
gh repo create @scope/project-mobile --template Syntek-Studio/mobile_template --private

# Option 2: Clone and re-initialise
git clone git@github.com:Syntek-Studio/mobile_template.git project-mobile
cd project-mobile
rm -rf .git
git init
git remote add origin git@github.com:Your-Org/project-mobile.git
```

### Post-Clone Setup

After cloning from template, the setup agent will:
1. **Detect the stack** from README.md (`React Native` + `GraphQL` → `stack-mobile`)
2. **Ask for project name** (e.g., `@company/project-mobile`)
3. **Update package.json** with new name and scope
4. **Update app.json** with bundle ID and package name
5. **Update .claude/CLAUDE.md** with project-specific details
6. **Verify git remote** matches project name

---

## Project CLAUDE.md

Create this file at `.claude/CLAUDE.md` or `CLAUDE.md` in the project root:

```markdown
# Project: [Insert Project Name] - Mobile

## Stack Overview

| Component | Technology |
|-----------|------------|
| **Type** | React Native Mobile (GraphQL Consumer) |
| **Language** | TypeScript 5.x |
| **Framework** | React Native / Expo |
| **Styling** | NativeWind + shared-lib design tokens |
| **GraphQL Client** | Apollo Client |
| **Code Generation** | GraphQL Code Generator |
| **Navigation** | React Navigation |
| **Testing** | Jest, React Native Testing Library |
| **Platform** | Native / Simulator (NO Docker) |

**CRITICAL:** Do NOT use Docker commands for this stack. Mobile runs natively.

---

## Skill Targets

- **Stack Skill:** `stack-mobile`
- **Global Skill:** `global-workflow`

---

## Architecture

This mobile app:
- **Consumes** GraphQL API from `stack-django` backend
- **Imports** styling and components from `stack-shared-lib`
- Shares the same design system with `stack-react` (React web)

---

## Environment

| Setting | Value |
|---------|-------|
| **Bundle ID (iOS)** | com.samdev.[projectname] |
| **Package Name (Android)** | com.samdev.[projectname] |
| **Backend API (Dev)** | http://localhost:8000/graphql/ |
| **Backend API (Staging)** | https://staging-api.[domain]/graphql/ |
| **Backend API (Production)** | https://api.[domain]/graphql/ |
| **Locale** | en_GB |
| **Timezone** | Europe/London |
| **Currency** | GBP (£) |

---

## Key Locations

| Directory | Purpose |
|-----------|---------|
| `src/screens/` | Screen components |
| `src/components/` | App-specific components |
| `src/navigation/` | Navigation configuration |
| `src/graphql/` | GraphQL queries, mutations, fragments |
| `src/graphql/generated/` | Auto-generated types and hooks |
| `src/hooks/` | Custom React hooks |
| `src/services/` | Non-GraphQL services |
| `src/utils/` | Utility functions |
| `ios/` | iOS native project |
| `android/` | Android native project |

---

## Development Commands

```bash
# Metro Bundler
npm start                            # Start Metro Bundler
npm start -- --reset-cache           # Start with cache reset

# iOS
npm run ios                          # Run on iOS Simulator
cd ios && pod install && cd ..       # Install iOS Pods

# Android
npm run android                      # Run on Android Emulator

# GraphQL
npm run codegen                      # Generate types from schema
npm run codegen:watch                # Watch mode

# Testing
npm test                             # Run all tests
npm test -- --coverage               # With coverage

# Linting
npm run lint                         # ESLint
npm run format                       # Prettier
```

---

## GraphQL Integration

### Apollo Client Setup
```typescript
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';
import AsyncStorage from '@react-native-async-storage/async-storage';

const httpLink = createHttpLink({
  uri: Config.GRAPHQL_ENDPOINT,
});

const authLink = setContext(async (_, { headers }) => {
  const token = await AsyncStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

export const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
});
```

---

## Shared Library Usage

Import components and styles from the shared library:

```typescript
import { Button, Card, Typography } from '@company/shared-lib';
import { colours, spacing } from '@company/shared-lib/tokens';
```

The shared library provides platform-agnostic:
- **Components:** Button, Card, Modal, Input, etc.
- **Typography:** Heading, Text, Label components
- **Design Tokens:** Colours, spacing, breakpoints
- **Fonts:** Pre-configured font families

---

## Code Conventions

### Components
- **Use React Native Core Components:** `<View>`, `<Text>`, `<ScrollView>`, `<TouchableOpacity>`
- **NEVER use HTML elements:** No `<div>`, `<span>`, `<p>`, `<button>`
- Use shared-lib components where available

### Styling
- Use NativeWind for Tailwind-like classes
- Design tokens from shared-lib for consistency
- Test on both iOS and Android

---

## Platform Considerations

### iOS Specific
- Handle safe area insets (`SafeAreaView`)
- Face ID / Touch ID authentication
- Test on notched devices (iPhone X+)

### Android Specific
- Handle back button navigation
- Status bar configuration
- Test on various Android versions

---

*(Add project-specific conventions here)*
```

---

## Settings File

Create this file at `.claude/settings.local.json`:

```json
{
  "language": "typescript",
  "framework": "react-native",
  "projectType": "graphql-consumer",
  "locale": "en_GB",
  "timezone": "Europe/London",
  "permissions": {
    "allow": [
      "Read(**)",
      "Edit(**)",
      "Write(**)",
      "Bash(npm:*)",
      "Bash(npx:*)",
      "Bash(pod:*)",
      "Bash(xcodebuild:*)",
      "Bash(gradle:*)",
      "Bash(adb:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo:*)",
      "Bash(docker:*)"
    ]
  },
  "environment": {
    "containerType": "none",
    "platform": "native",
    "envFiles": [
      ".env",
      ".env.development",
      ".env.staging",
      ".env.production"
    ]
  },
  "testing": {
    "framework": "jest",
    "command": "npm test",
    "coverageCommand": "npm test -- --coverage"
  },
  "linting": {
    "eslint": "npm run lint",
    "prettier": "npm run format"
  },
  "graphql": {
    "backend": "stack-django",
    "endpoint": "GRAPHQL_ENDPOINT",
    "codegen": "npm run codegen"
  },
  "dependencies": {
    "sharedLib": "@company/shared-lib",
    "backend": "stack-django"
  }
}
```

---

## Commands

Create these files in `.claude/commands/`:

**CRITICAL:** Commands should reference the root-level environment scripts (`dev.sh`, `test.sh`, `staging.sh`, `production.sh`) to ensure consistency and avoid duplication.

### .claude/commands/dev.md

```markdown
---
description: Start the Metro bundler and run the app
usage: /dev
---

Start the React Native development environment.

**Run:** `./dev.sh`

**Prerequisites:**
- Ensure Django backend is running at http://localhost:8000
- Ensure shared-lib is linked: `npm link @company/shared-lib`

This script will:
1. Install dependencies if needed
2. Install iOS Pods if needed
3. Start Metro Bundler

Then run `npm run ios` or `npm run android` in another terminal.

$ARGUMENTS
```

### .claude/commands/test.md

```markdown
---
description: Run the Jest test suite
usage: /test
---

Run the project's Jest test suite.

**Run:** `./test.sh`

This script will run the full test suite.

**Additional Commands:**
- Watch mode: `npm test -- --watch`
- With coverage: `npm test -- --coverage`
- Single file: `npm test -- $ARGUMENTS`

$ARGUMENTS
```

### .claude/commands/staging.md

```markdown
---
description: Build for staging environment
usage: /staging
---

Build the mobile app for staging.

**Run:** `./staging.sh`

This script will:
1. Run the test suite first
2. Generate GraphQL types
3. Build iOS app with staging scheme
4. Build Android app with staging flavour

**Output:**
- iOS: `ios/build/`
- Android: `android/app/build/outputs/`

$ARGUMENTS
```

### .claude/commands/production.md

```markdown
---
description: Build for production environment
usage: /production
---

Build the mobile app for production.

**Run:** `./production.sh`

This script will:
1. Prompt for confirmation (safety check)
2. Run the test suite first
3. Generate GraphQL types
4. Build iOS app with production scheme
5. Build Android AAB for Play Store

**Output:**
- iOS: `ios/build/`
- Android: `android/app/build/outputs/`

$ARGUMENTS
```

### .claude/commands/codegen.md

```markdown
---
description: Generate TypeScript types from GraphQL schema
usage: /codegen
---

Generate TypeScript types and hooks from the Django backend's GraphQL schema.

**Commands:**
- One-time: `npm run codegen`
- Watch mode: `npm run codegen:watch`

**Prerequisites:**
- Django backend must be running with introspection enabled
- Schema endpoint: http://localhost:8000/graphql/

**Output:** `src/graphql/generated/`

$ARGUMENTS
```

### .claude/commands/build-ios.md

```markdown
---
description: Build iOS app for release
usage: /build-ios
---

Build the iOS application for release.

**Run:** `./production.sh` (recommended for full build)

Or manually:
1. Ensure provisioning profiles are configured
2. Run `cd ios && pod install && cd ..`
3. Open Xcode: `open ios/[ProjectName].xcworkspace`
4. Select "Any iOS Device" as target
5. Product > Archive
6. Distribute App through App Store Connect

$ARGUMENTS
```

### .claude/commands/build-android.md

```markdown
---
description: Build Android APK/AAB for release
usage: /build-android
---

Build the Android application for release.

**Run:** `./production.sh` (recommended for full build)

Or manually:
1. Ensure signing key is configured in `android/app/build.gradle`
2. Run `cd android && ./gradlew assembleRelease && cd ..` for APK
3. Run `cd android && ./gradlew bundleRelease && cd ..` for AAB
4. Find output in `android/app/build/outputs/`

$ARGUMENTS
```

### .claude/commands/link-shared.md

```markdown
---
description: Link the shared library for local development
usage: /link-shared
---

Link the shared library for local development.

**Using npm link:**
```bash
# In shared-lib project:
npm link

# In this project:
npm link @company/shared-lib
```

**Using yalc (recommended):**
```bash
# In shared-lib project:
yalc push

# In this project:
yalc add @company/shared-lib
```

**After linking, rebuild:**
```bash
cd ios && pod install && cd ..
npm start -- --reset-cache
```

$ARGUMENTS
```

### .claude/commands/clean.md

```markdown
---
description: Clean build caches and reinstall dependencies
usage: /clean
---

Clean all build caches and reinstall dependencies.

**Steps:**
1. `rm -rf node_modules`
2. `npm install`
3. `cd ios && rm -rf build Pods && pod install && cd ..`
4. `cd android && ./gradlew clean && cd ..`
5. `npm start -- --reset-cache`

$ARGUMENTS
```

---

## Directory Structure

```
[project-root]/
├── .claude/
│   ├── CLAUDE.md
│   ├── settings.local.json
│   └── commands/
│       ├── dev.md              # References ./dev.sh
│       ├── test.md             # References ./test.sh
│       ├── staging.md          # References ./staging.sh
│       ├── production.md       # References ./production.sh
│       ├── codegen.md
│       ├── build-ios.md
│       ├── build-android.md
│       ├── link-shared.md
│       └── clean.md
├── src/
│   ├── screens/
│   │   ├── HomeScreen.tsx
│   │   └── ProfileScreen.tsx
│   ├── components/                 # App-specific components
│   │   ├── layout/
│   │   │   └── Header.tsx
│   │   └── features/
│   │       └── UserCard.tsx
│   ├── navigation/
│   │   ├── types.ts
│   │   ├── RootNavigator.tsx
│   │   └── TabNavigator.tsx
│   ├── graphql/
│   │   ├── queries/
│   │   │   ├── user.graphql
│   │   │   └── content.graphql
│   │   ├── mutations/
│   │   │   └── auth.graphql
│   │   ├── fragments/
│   │   │   └── userFields.graphql
│   │   └── generated/              # Auto-generated by codegen
│   │       ├── types.ts
│   │       └── hooks.ts
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   └── useCurrentUser.ts
│   ├── services/
│   │   └── apollo.ts               # Apollo Client setup
│   ├── utils/
│   │   └── formatters.ts
│   ├── types/
│   │   └── index.ts
│   ├── assets/
│   │   ├── images/
│   │   └── fonts/
│   └── App.tsx
├── ios/
│   ├── [ProjectName]/
│   ├── [ProjectName].xcworkspace
│   └── Podfile
├── android/
│   ├── app/
│   └── build.gradle
├── __tests__/
├── docs/
│   ├── API/
│   ├── ARCHITECTURE/
│   ├── BUGS/
│   ├── DEVOPS/
│   ├── GUIDES/
│   ├── METRICS/               # Self-learning system data
│   │   ├── README.md
│   │   ├── config.json
│   │   ├── runs/
│   │   ├── feedback/
│   │   ├── aggregates/
│   │   ├── variants/
│   │   └── optimisations/
│   ├── PLANS/
│   ├── QA/
│   ├── STORIES/
│   └── TESTS/
├── package.json
├── tsconfig.json
├── babel.config.js
├── metro.config.js
├── tailwind.config.js
├── codegen.ts                      # GraphQL Code Generator config
├── jest.config.js
├── .eslintrc.js
├── .prettierrc
├── app.json
├── .env.example
├── .env.development.example
├── .env.staging.example
├── .env.production.example
├── CHANGELOG.md
├── README.md
├── dev.sh
├── test.sh
├── staging.sh
└── production.sh
```

---

## Environment-Specific Command Files

**CRITICAL:** All React Native projects MUST have environment-specific command files for managing different environments.

**Note:** Mobile projects run natively (NOT in Docker), so these scripts use npm directly.

### Required Root-Level Scripts

| File | Purpose | Environment |
|------|---------|-------------|
| `dev.sh` | Start development environment | Development |
| `test.sh` | Run test suite | Testing |
| `staging.sh` | Build for staging | Staging |
| `production.sh` | Build for production | Production |

### dev.sh

```bash
#!/bin/bash
set -e

echo "🚀 Starting development environment..."

# Install dependencies if needed
if [ ! -d "node_modules" ]; then
    npm install
fi

# Install iOS pods if needed
if [ -d "ios" ] && [ ! -d "ios/Pods" ]; then
    echo "📦 Installing iOS Pods..."
    cd ios && pod install && cd ..
fi

# Start Metro bundler
echo "✅ Starting Metro bundler..."
echo "📱 Run 'npm run ios' or 'npm run android' in another terminal"
npm start
```

### test.sh

```bash
#!/bin/bash
set -e

echo "🧪 Running test suite..."

npm test

echo "✅ Tests complete!"
```

### staging.sh

```bash
#!/bin/bash
set -e

echo "🚀 Building for staging..."

# Run tests first
./test.sh

# Generate GraphQL types
npm run codegen

echo "📱 Building iOS..."
cd ios && xcodebuild -workspace *.xcworkspace -scheme "*-Staging" -configuration Release clean archive && cd ..

echo "🤖 Building Android..."
cd android && ./gradlew assembleStaging && cd ..

echo "✅ Staging builds complete!"
echo "📁 iOS: ios/build/"
echo "📁 Android: android/app/build/outputs/"
```

### production.sh

```bash
#!/bin/bash
set -e

echo "🚀 Building for production..."

read -p "⚠️  Are you sure you want to build for PRODUCTION? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "❌ Build cancelled."
    exit 1
fi

# Run tests first
./test.sh

# Generate GraphQL types
npm run codegen

echo "📱 Building iOS..."
cd ios && xcodebuild -workspace *.xcworkspace -scheme "*-Production" -configuration Release clean archive && cd ..

echo "🤖 Building Android..."
cd android && ./gradlew bundleRelease && cd ..

echo "✅ Production builds complete!"
echo "📁 iOS: ios/build/"
echo "📁 Android: android/app/build/outputs/"
```

### Command File Permissions

**CRITICAL:** All shell scripts MUST be executable:

```bash
chmod +x dev.sh test.sh staging.sh production.sh
```

---

## GraphQL Code Generator Configuration

Create `codegen.ts`:

```typescript
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: process.env.GRAPHQL_ENDPOINT || 'http://localhost:8000/graphql/',
  documents: ['src/graphql/**/*.graphql'],
  generates: {
    'src/graphql/generated/types.ts': {
      plugins: ['typescript'],
    },
    'src/graphql/generated/hooks.ts': {
      preset: 'client',
      plugins: ['typescript-operations', 'typescript-react-apollo'],
      config: {
        withHooks: true,
        withComponent: false,
      },
    },
  },
};

export default config;
```

---

## Environment Variables

Use `react-native-config` for environment variables:

```env
# .env.development.example
GRAPHQL_ENDPOINT=http://localhost:8000/graphql/
APP_NAME="[Project Name] (Development)"

# .env.staging.example
GRAPHQL_ENDPOINT=https://staging-api.[domain]/graphql/
APP_NAME="[Project Name] (Staging)"

# .env.production.example
GRAPHQL_ENDPOINT=https://api.[domain]/graphql/
APP_NAME="[Project Name]"
```

---

## Package Dependencies

Key dependencies to add:

```json
{
  "dependencies": {
    "@apollo/client": "^3.8.0",
    "@company/shared-lib": "^1.0.0",
    "@react-navigation/native": "^6.1.0",
    "@react-navigation/native-stack": "^6.9.0",
    "@react-native-async-storage/async-storage": "^1.21.0",
    "graphql": "^16.8.0",
    "nativewind": "^2.0.0",
    "react-native-config": "^1.5.0",
    "react-native-safe-area-context": "^4.8.0",
    "react-native-screens": "^3.29.0"
  },
  "devDependencies": {
    "@graphql-codegen/cli": "^5.0.0",
    "@graphql-codegen/client-preset": "^4.1.0",
    "@graphql-codegen/typescript": "^4.0.0",
    "@graphql-codegen/typescript-operations": "^4.0.0",
    "@graphql-codegen/typescript-react-apollo": "^4.0.0",
    "tailwindcss": "^3.4.0"
  }
}
```
