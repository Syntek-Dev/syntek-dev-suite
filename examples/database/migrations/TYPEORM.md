# TypeORM Migrations

## Metadata

| Property         | Value         |
| ---------------- | ------------- |
| **Version**      | 2.0.0         |
| **Last Updated** | December 2025 |
| **Status**       | Stable        |

## Framework Versions Tested

| Framework  | Version  | Tested Date |
| ---------- | -------- | ----------- |
| TypeORM    | 0.3.x    | 20/12/2025  |
| NestJS     | 10.x     | 20/12/2025  |
| Node.js    | 24.x LTS | 20/12/2025  |
| TypeScript | 5.9      | 20/12/2025  |
| PostgreSQL | 18.x     | 20/12/2025  |

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Table of Contents](#table-of-contents)
- [Basic Entity](#basic-entity)
- [Entity with Relations](#entity-with-relations)
- [PII Entity (GDPR Compliant)](#pii-entity-gdpr-compliant)
- [Role-Based Access Control Entities](#role-based-access-control-entities)
- [Migration File Example](#migration-file-example)
- [Running Migrations](#running-migrations)
- [Data Source Configuration](#data-source-configuration)


## Basic Entity

```typescript
// src/entities/user.entity.ts

/**
 * User entity for authentication and authorisation.
 *
 * Uses UUID for public-facing identifiers and soft deletes
 * for GDPR-compliant data retention.
 */

import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  Index,
  OneToMany,
  OneToOne,
} from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'uuid', generated: 'uuid', unique: true })
  @Index()
  uuid: string;

  @Column({ type: 'varchar', length: 150, unique: true })
  @Index()
  username: string;

  @Column({ type: 'varchar', length: 255, unique: true })
  @Index()
  email: string;

  @Column({ type: 'timestamp', nullable: true, name: 'email_verified_at' })
  emailVerifiedAt: Date | null;

  @Column({ type: 'varchar', length: 255 })
  password: string;

  @Column({ type: 'boolean', default: true, name: 'is_active' })
  isActive: boolean;

  @Column({ type: 'boolean', default: false, name: 'is_staff' })
  isStaff: boolean;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  @DeleteDateColumn({ name: 'deleted_at' })
  deletedAt: Date | null;

  // Relations
  @OneToMany(() => Order, (order) => order.user)
  orders: Order[];

  @OneToMany(() => UserRole, (userRole) => userRole.user)
  userRoles: UserRole[];

  @OneToOne(() => UserPii, (pii) => pii.user)
  pii: UserPii;
}
```

## Entity with Relations

```typescript
// src/entities/order.entity.ts

/**
 * Order entity representing a customer order.
 *
 * Uses UUID for public-facing order reference.
 */

import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  ManyToOne,
  OneToMany,
  JoinColumn,
  Index,
} from 'typeorm';
import { User } from './user.entity';
import { OrderItem } from './order-item.entity';

export enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled',
}

@Entity('orders')
@Index(['userId', 'status'])
@Index(['createdAt'])
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'uuid', generated: 'uuid', unique: true })
  @Index()
  uuid: string;

  @Column({ name: 'user_id' })
  userId: number;

  @Column({
    type: 'enum',
    enum: OrderStatus,
    default: OrderStatus.PENDING,
  })
  status: OrderStatus;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  total: number;

  @Column({ type: 'jsonb', nullable: true })
  metadata: Record<string, any> | null;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  @DeleteDateColumn({ name: 'deleted_at' })
  deletedAt: Date | null;

  // Relations
  @ManyToOne(() => User, (user) => user.orders, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: User;

  @OneToMany(() => OrderItem, (item) => item.order)
  items: OrderItem[];
}
```

## PII Entity (GDPR Compliant)

```typescript
// src/entities/user-pii.entity.ts

/**
 * UserPii entity for storing user PII separately for GDPR compliance.
 *
 * Uses HMAC-SHA256 hashes for lookups (irreversible)
 * and encrypted values for storage (reversible).
 */

import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToOne,
  JoinColumn,
  Index,
} from 'typeorm';
import { User } from './user.entity';

@Entity('user_pii')
export class UserPii {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ name: 'user_id', unique: true })
  userId: number;

  // Hashed columns for lookup (64 chars for SHA256)
  @Column({ type: 'varchar', length: 64, name: 'email_hash' })
  @Index()
  emailHash: string;

  @Column({ type: 'varchar', length: 64, nullable: true, name: 'phone_hash' })
  @Index()
  phoneHash: string | null;

  // Encrypted columns for storage
  @Column({ type: 'text', name: 'email_encrypted' })
  emailEncrypted: string;

  @Column({ type: 'text', nullable: true, name: 'phone_encrypted' })
  phoneEncrypted: string | null;

  @Column({ type: 'text', name: 'full_name_encrypted' })
  fullNameEncrypted: string;

  @Column({ type: 'text', nullable: true, name: 'address_encrypted' })
  addressEncrypted: string | null;

  @Column({ type: 'text', nullable: true, name: 'dob_encrypted' })
  dobEncrypted: string | null;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  // Relations
  @OneToOne(() => User, (user) => user.pii, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: User;
}
```

## Role-Based Access Control Entities

```typescript
// src/entities/role.entity.ts

/**
 * Role entity for RBAC.
 */

import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
} from 'typeorm';
import { RolePermission } from './role-permission.entity';
import { UserRole } from './user-role.entity';

@Entity('roles')
export class Role {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'varchar', length: 50, unique: true })
  name: string;

  @Column({ type: 'varchar', length: 100, nullable: true, name: 'display_name' })
  displayName: string | null;

  @Column({ type: 'text', nullable: true })
  description: string | null;

  @Column({ type: 'boolean', default: false, name: 'is_system' })
  isSystem: boolean;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  // Relations
  @OneToMany(() => RolePermission, (rp) => rp.role)
  rolePermissions: RolePermission[];

  @OneToMany(() => UserRole, (ur) => ur.role)
  userRoles: UserRole[];
}

// src/entities/permission.entity.ts

/**
 * Permission entity for granular access control.
 */

import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
  Index,
} from 'typeorm';
import { RolePermission } from './role-permission.entity';

@Entity('permissions')
export class Permission {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'varchar', length: 100, unique: true })
  name: string;

  @Column({ type: 'varchar', length: 100, nullable: true, name: 'display_name' })
  displayName: string | null;

  @Column({ type: 'text', nullable: true })
  description: string | null;

  @Column({ type: 'varchar', length: 50, nullable: true, name: 'group_name' })
  @Index()
  groupName: string | null;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  // Relations
  @OneToMany(() => RolePermission, (rp) => rp.permission)
  rolePermissions: RolePermission[];
}

// src/entities/role-permission.entity.ts

/**
 * RolePermission entity linking roles to permissions.
 */

import { Entity, PrimaryColumn, ManyToOne, JoinColumn } from 'typeorm';
import { Role } from './role.entity';
import { Permission } from './permission.entity';

@Entity('role_permissions')
export class RolePermission {
  @PrimaryColumn({ name: 'role_id' })
  roleId: number;

  @PrimaryColumn({ name: 'permission_id' })
  permissionId: number;

  @ManyToOne(() => Role, (role) => role.rolePermissions, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'role_id' })
  role: Role;

  @ManyToOne(() => Permission, (permission) => permission.rolePermissions, {
    onDelete: 'CASCADE',
  })
  @JoinColumn({ name: 'permission_id' })
  permission: Permission;
}

// src/entities/user-role.entity.ts

/**
 * UserRole entity linking users to roles with metadata.
 */

import {
  Entity,
  PrimaryColumn,
  Column,
  ManyToOne,
  JoinColumn,
  CreateDateColumn,
} from 'typeorm';
import { User } from './user.entity';
import { Role } from './role.entity';

@Entity('user_roles')
export class UserRole {
  @PrimaryColumn({ name: 'user_id' })
  userId: number;

  @PrimaryColumn({ name: 'role_id' })
  roleId: number;

  @CreateDateColumn({ name: 'assigned_at' })
  assignedAt: Date;

  @Column({ name: 'assigned_by', nullable: true })
  assignedBy: number | null;

  @ManyToOne(() => User, (user) => user.userRoles, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: User;

  @ManyToOne(() => Role, (role) => role.userRoles, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'role_id' })
  role: Role;
}
```

## Migration File Example

```typescript
// src/migrations/1703073600000-InitialSchema.ts

import { MigrationInterface, QueryRunner } from 'typeorm';

/**
 * Initial schema migration creating users, orders, and RBAC tables.
 */
export class InitialSchema1703073600000 implements MigrationInterface {
  name = 'InitialSchema1703073600000';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create users table
    await queryRunner.query(`
      CREATE TABLE "users" (
        "id" SERIAL PRIMARY KEY,
        "uuid" uuid NOT NULL DEFAULT uuid_generate_v4(),
        "username" varchar(150) NOT NULL UNIQUE,
        "email" varchar(255) NOT NULL UNIQUE,
        "email_verified_at" timestamp,
        "password" varchar(255) NOT NULL,
        "is_active" boolean NOT NULL DEFAULT true,
        "is_staff" boolean NOT NULL DEFAULT false,
        "created_at" timestamp NOT NULL DEFAULT now(),
        "updated_at" timestamp NOT NULL DEFAULT now(),
        "deleted_at" timestamp
      )
    `);

    await queryRunner.query(`CREATE INDEX "IDX_users_uuid" ON "users" ("uuid")`);
    await queryRunner.query(`CREATE INDEX "IDX_users_email" ON "users" ("email")`);
    await queryRunner.query(`CREATE INDEX "IDX_users_username" ON "users" ("username")`);

    // Create orders table
    await queryRunner.query(`
      CREATE TYPE "order_status" AS ENUM (
        'pending', 'processing', 'shipped', 'delivered', 'cancelled'
      )
    `);

    await queryRunner.query(`
      CREATE TABLE "orders" (
        "id" SERIAL PRIMARY KEY,
        "uuid" uuid NOT NULL DEFAULT uuid_generate_v4(),
        "user_id" integer NOT NULL REFERENCES "users"("id") ON DELETE CASCADE,
        "status" order_status NOT NULL DEFAULT 'pending',
        "total" decimal(10, 2) NOT NULL,
        "metadata" jsonb,
        "created_at" timestamp NOT NULL DEFAULT now(),
        "updated_at" timestamp NOT NULL DEFAULT now(),
        "deleted_at" timestamp
      )
    `);

    await queryRunner.query(`CREATE INDEX "IDX_orders_user_status" ON "orders" ("user_id", "status")`);
    await queryRunner.query(`CREATE INDEX "IDX_orders_created_at" ON "orders" ("created_at")`);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "orders"`);
    await queryRunner.query(`DROP TYPE "order_status"`);
    await queryRunner.query(`DROP TABLE "users"`);
  }
}
```

## Running Migrations

```bash
# Generate migration from entity changes
npx typeorm-ts-node-commonjs migration:generate src/migrations/MigrationName -d src/data-source.ts

# Create empty migration
npx typeorm-ts-node-commonjs migration:create src/migrations/MigrationName

# Run pending migrations
npx typeorm-ts-node-commonjs migration:run -d src/data-source.ts

# Revert last migration
npx typeorm-ts-node-commonjs migration:revert -d src/data-source.ts

# Show migrations status
npx typeorm-ts-node-commonjs migration:show -d src/data-source.ts

# Synchronise schema (development only!)
npx typeorm-ts-node-commonjs schema:sync -d src/data-source.ts
```

## Data Source Configuration

```typescript
// src/data-source.ts

import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';
import { Order } from './entities/order.entity';
import { UserPii } from './entities/user-pii.entity';
import { Role } from './entities/role.entity';
import { Permission } from './entities/permission.entity';
import { RolePermission } from './entities/role-permission.entity';
import { UserRole } from './entities/user-role.entity';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT || '5432'),
  username: process.env.DB_USER || 'postgres',
  password: process.env.DB_PASSWORD || 'password',
  database: process.env.DB_NAME || 'app_dev',
  entities: [User, Order, UserPii, Role, Permission, RolePermission, UserRole],
  migrations: ['src/migrations/*.ts'],
  synchronize: false, // Never true in production
  logging: process.env.NODE_ENV === 'development',
});
```
