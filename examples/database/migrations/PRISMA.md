# Prisma Migrations

## Metadata

| Property         | Value   |
| ---------------- | ------- |
| **Version**      | 2.0.0   |
| **Last Updated** | 12/2025 |
| **Status**       | Stable  |

## Framework Versions Tested

| Framework  | Version | Tested Date |
| ---------- | ------- | ----------- |
| Prisma     | 6.x     | 20/12/2025  |
| Node.js    | 24.x    | 20/12/2025  |
| TypeScript | 5.9     | 20/12/2025  |
| PostgreSQL | 18.x    | 20/12/2025  |

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Table of Contents](#table-of-contents)
- [Basic Schema](#basic-schema)
- [Schema with Relations](#schema-with-relations)
- [PII Schema (GDPR Compliant)](#pii-schema-gdpr-compliant)
- [Role-Based Access Control Schema](#role-based-access-control-schema)
- [Running Migrations](#running-migrations)
- [Seed Script](#seed-script)


## Basic Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

/// User account for authentication and authorisation.
/// Uses UUID for public-facing identifiers.
model User {
  id              Int       @id @default(autoincrement())
  uuid            String    @unique @default(uuid())
  username        String    @unique @db.VarChar(150)
  email           String    @unique @db.VarChar(255)
  emailVerifiedAt DateTime? @map("email_verified_at")
  password        String    @db.VarChar(255)
  isActive        Boolean   @default(true) @map("is_active")
  isStaff         Boolean   @default(false) @map("is_staff")
  createdAt       DateTime  @default(now()) @map("created_at")
  updatedAt       DateTime  @updatedAt @map("updated_at")
  deletedAt       DateTime? @map("deleted_at")

  // Relations
  orders          Order[]
  userRoles       UserRole[]
  pii             UserPii?

  @@index([email])
  @@index([username])
  @@map("users")
}
```

## Schema with Relations

```prisma
/// Order status enumeration.
enum OrderStatus {
  pending
  processing
  shipped
  delivered
  cancelled
}

/// Represents a customer order.
/// Uses UUID for public-facing order reference.
model Order {
  id        Int         @id @default(autoincrement())
  uuid      String      @unique @default(uuid())
  userId    Int         @map("user_id")
  status    OrderStatus @default(pending)
  total     Decimal     @db.Decimal(10, 2)
  metadata  Json?
  createdAt DateTime    @default(now()) @map("created_at")
  updatedAt DateTime    @updatedAt @map("updated_at")
  deletedAt DateTime?   @map("deleted_at")

  // Relations
  user      User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  items     OrderItem[]

  @@index([userId, status])
  @@index([createdAt])
  @@map("orders")
}

/// Individual item within an order.
model OrderItem {
  id        Int      @id @default(autoincrement())
  orderId   Int      @map("order_id")
  productId Int      @map("product_id")
  quantity  Int
  unitPrice Decimal  @map("unit_price") @db.Decimal(10, 2)
  createdAt DateTime @default(now()) @map("created_at")

  // Relations
  order     Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product   Product  @relation(fields: [productId], references: [id], onDelete: Restrict)

  @@map("order_items")
}

/// Product catalogue item.
model Product {
  id          Int         @id @default(autoincrement())
  uuid        String      @unique @default(uuid())
  sku         String      @unique @db.VarChar(100)
  name        String      @db.VarChar(255)
  description String?     @db.Text
  price       Decimal     @db.Decimal(10, 2)
  isActive    Boolean     @default(true) @map("is_active")
  createdAt   DateTime    @default(now()) @map("created_at")
  updatedAt   DateTime    @updatedAt @map("updated_at")

  // Relations
  orderItems  OrderItem[]

  @@index([sku])
  @@map("products")
}
```

## PII Schema (GDPR Compliant)

```prisma
/// Stores user PII separately for GDPR compliance.
/// Uses HMAC-SHA256 hashes for lookups, encrypted values for storage.
model UserPii {
  id              Int      @id @default(autoincrement())
  userId          Int      @unique @map("user_id")

  // Hashed columns for lookup (irreversible, 64 chars for SHA256)
  emailHash       String   @map("email_hash") @db.VarChar(64)
  phoneHash       String?  @map("phone_hash") @db.VarChar(64)

  // Encrypted columns for storage (reversible)
  emailEncrypted  String   @map("email_encrypted") @db.Text
  phoneEncrypted  String?  @map("phone_encrypted") @db.Text
  fullNameEncrypted String @map("full_name_encrypted") @db.Text
  addressEncrypted String? @map("address_encrypted") @db.Text
  dobEncrypted    String?  @map("dob_encrypted") @db.Text

  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")

  // Relations
  user            User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([emailHash])
  @@index([phoneHash])
  @@map("user_pii")
}
```

## Role-Based Access Control Schema

```prisma
/// Role definition for RBAC.
model Role {
  id          Int      @id @default(autoincrement())
  name        String   @unique @db.VarChar(50)
  displayName String?  @map("display_name") @db.VarChar(100)
  description String?  @db.Text
  isSystem    Boolean  @default(false) @map("is_system")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  // Relations
  rolePermissions RolePermission[]
  userRoles       UserRole[]

  @@map("roles")
}

/// Permission definition for granular access control.
model Permission {
  id          Int      @id @default(autoincrement())
  name        String   @unique @db.VarChar(100)
  displayName String?  @map("display_name") @db.VarChar(100)
  description String?  @db.Text
  groupName   String?  @map("group_name") @db.VarChar(50)
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  // Relations
  rolePermissions RolePermission[]

  @@index([groupName])
  @@map("permissions")
}

/// Links roles to permissions (many-to-many).
model RolePermission {
  roleId       Int @map("role_id")
  permissionId Int @map("permission_id")

  // Relations
  role       Role       @relation(fields: [roleId], references: [id], onDelete: Cascade)
  permission Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)

  @@id([roleId, permissionId])
  @@map("role_permissions")
}

/// Links users to roles with assignment metadata.
model UserRole {
  userId     Int      @map("user_id")
  roleId     Int      @map("role_id")
  assignedAt DateTime @default(now()) @map("assigned_at")
  assignedBy Int?     @map("assigned_by")

  // Relations
  user       User @relation(fields: [userId], references: [id], onDelete: Cascade)
  role       Role @relation(fields: [roleId], references: [id], onDelete: Cascade)

  @@id([userId, roleId])
  @@map("user_roles")
}
```

## Running Migrations

```bash
# Generate migration from schema changes
npx prisma migrate dev --name init

# Apply migrations to database
npx prisma migrate dev

# Apply migrations in production (no prompts)
npx prisma migrate deploy

# Reset database (drops all data!)
npx prisma migrate reset

# View migration status
npx prisma migrate status

# Generate Prisma Client
npx prisma generate

# Push schema without creating migration (dev only)
npx prisma db push

# Pull existing database schema
npx prisma db pull

# Format schema file
npx prisma format
```

## Seed Script

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

/**
 * Seeds the database with initial data.
 * Creates roles, permissions, and an admin user.
 */
async function main(): Promise<void> {
  // Create permissions
  const permissions = await Promise.all([
    prisma.permission.create({
      data: { name: 'pii.access', displayName: 'Access PII', groupName: 'PII' },
    }),
    prisma.permission.create({
      data: { name: 'pii.export', displayName: 'Export PII', groupName: 'PII' },
    }),
    prisma.permission.create({
      data: { name: 'pii.delete', displayName: 'Delete PII', groupName: 'PII' },
    }),
    prisma.permission.create({
      data: { name: 'users.view', displayName: 'View Users', groupName: 'Users' },
    }),
    prisma.permission.create({
      data: { name: 'users.edit', displayName: 'Edit Users', groupName: 'Users' },
    }),
  ]);

  // Create admin role
  const adminRole = await prisma.role.create({
    data: {
      name: 'admin',
      displayName: 'Administrator',
      description: 'Full system access',
      isSystem: true,
    },
  });

  // Assign all permissions to admin role
  await Promise.all(
    permissions.map((permission) =>
      prisma.rolePermission.create({
        data: {
          roleId: adminRole.id,
          permissionId: permission.id,
        },
      })
    )
  );

  console.log('Database seeded successfully');
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Add to `package.json`:

```json
{
  "prisma": {
    "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  }
}
```

Run with:

```bash
npx prisma db seed
```
