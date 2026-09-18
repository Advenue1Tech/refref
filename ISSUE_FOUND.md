# RefRef Open-Source Issues & Solutions

This document documents technical issues encountered when running the RefRef open-source platform out-of-the-box, along with step-by-step solutions to reproduce, resolve, and clean up the database for production.

---

## 1. `DATABASE_URL` Environment Variable Error in `db:seed`

### ❌ Problem Description
Executing `pnpm -F @refref/coredb db:seed` fails out-of-the-box with the following error:

```
Error: DATABASE_URL environment variable is required
    at <anonymous> (packages/coredb/src/seed.ts:7:9)
```

In `packages/coredb/src/seed.ts`, the code strictly requires `process.env.DATABASE_URL` to be set before execution:

```typescript
// Get database URL from environment
const DATABASE_URL = process.env.DATABASE_URL;
if (!DATABASE_URL) {
  throw new Error("DATABASE_URL environment variable is required");
}
```

### 🛠 Solutions

#### Solution A: Pass `DATABASE_URL` via Command Line (Recommended for Clean Clones)
Run `db:seed` by explicitly passing the connection string in your terminal shell without modifying source files:

* **PowerShell:**
  ```powershell
  $env:DATABASE_URL="postgresql://postgres:postgres@localhost:5432/refref"; pnpm -F @refref/coredb db:seed
  ```
* **CMD (Windows Command Prompt):**
  ```cmd
  set DATABASE_URL=postgresql://postgres:postgres@localhost:5432/refref && pnpm -F @refref/coredb db:seed
  ```
* **Bash / Linux / macOS:**
  ```bash
  DATABASE_URL="postgresql://postgres:postgres@localhost:5432/refref" pnpm -F @refref/coredb db:seed
  ```

#### Solution B: Load Environment Variables from `.env` in `packages/coredb/src/seed.ts`
To avoid hardcoding sensitive database credentials inside source files, configure `seed.ts` to load environment variables from `apps/webapp/.env`:

```typescript
import dotenv from "dotenv";
import path from "path";

// Load environment variables from apps/webapp/.env if DATABASE_URL is not set
if (!process.env.DATABASE_URL) {
  dotenv.config({ path: path.resolve(process.cwd(), "../../apps/webapp/.env") });
}

const DATABASE_URL = process.env.DATABASE_URL;
if (!DATABASE_URL) {
  throw new Error("DATABASE_URL environment variable is required. Ensure apps/webapp/.env is populated or set DATABASE_URL in your shell.");
}
```

---

## 2. Seeded Users Cannot Log In Out-of-the-Box

### ❌ Problem Description
Running `pnpm -F @refref/coredb db:seed` populates the `user` table with sample users (`test1@test.com`, `test2@test.com`, `test3@test.com`), but **does not insert any records into the `account` table**.

RefRef uses **Better-Auth**. To authenticate via password, Better-Auth requires:
1. A row in the `user` table.
2. A matching row in the `account` table containing `provider_id = 'credential'` and a valid hashed `password` linked to `user_id`.

Because the `account` table remains empty after running `db:seed`, signing in as any seeded user fails.

---

### 🛠 Solutions

#### Solution A: Normal Web UI Signup
Navigate to `http://refref-webapp.localhost:1355/auth/sign-up` in your browser and register a new user manually.

* **SMTP Requirement Explanation:**
  * **Password Sign-Up:** Does **NOT** require an SMTP server or Resend API key in development. Better-Auth creates the `user` and `account` records directly in PostgreSQL and authenticates the session immediately.
  * **Magic Links / Email Verification:** **Does** require an SMTP provider or Resend API key (`RESEND_API_KEY`). Without SMTP, magic link emails cannot be delivered.

---

#### Solution B: Seed the `account` Table for Seeded Users
To allow seeded users (`test1@test.com`, `test2@test.com`, `test3@test.com`) to log in out-of-the-box (e.g. using password `Password123!`) without altering their fixed seed IDs and organization ownerships:

Add password hash generation and `account` insertion to `packages/coredb/src/seed.ts` inside `seedData()`:

```typescript
import { hashPassword } from "better-auth/crypto";

// Inside seedData function after creating users:
console.log("🔐 Creating accounts with passwords...");
const hashedPassword = await hashPassword("Password123!");

const ACCOUNTS = SEED_DATA.USERS.map((user) => ({
  id: `acc_${user.id}`,
  accountId: user.id,
  providerId: "credential",
  userId: user.id,
  password: hashedPassword,
}));

await tx.insert(schema.account).values(ACCOUNTS).onConflictDoNothing();
```

---

#### 🧹 Removing Seeded Users for Production
Before deploying to production, all seeded test users, sample organizations, and template data should be wiped from the database.

* **Command to remove seeded data:**
  ```bash
  pnpm -F @refref/coredb db:deleteseed
  ```
  *(This executes `tsx src/seed.ts delete`, which cleans out all mock users, organizations, products, programs, and rewards).*
