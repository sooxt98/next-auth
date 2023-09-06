---
id: prisma
title: Prisma Adapter
---

# Prisma

You can also use NextAuth.js with the new experimental Adapter for [Prisma](https://www.prisma.io/docs/). This version of the Prisma Adapter is not included in the core `next-auth` package, and must be installed separately.

:::info
You may have noticed there is a `prisma` and `prisma-legacy` adapter. This is due to historical reasons, but the code has mostly converged so that there is no longer much difference between the two. The legacy adapter, however, does have the ability to rename tables which the newer version does not.
:::

To use this Adapter, you need to install Prisma Client, Prisma CLI, and the separate `@next-auth/prisma-adapter@canary` package:

```
npm install @prisma/client @next-auth/prisma-adapter@canary
npm install prisma --save-dev
```

Configure your NextAuth.js to use the Prisma Adapter:

```javascript title="pages/api/auth/[...nextauth].js"
import NextAuth from "next-auth"
import Providers from "next-auth/providers"
import { PrismaAdapter } from "@next-auth/prisma-adapter"
import { PrismaClient } from "@prisma/client"

const prisma = new PrismaClient()

export default NextAuth({
  providers: [
    Providers.Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
  ],
  adapter: PrismaAdapter(prisma),
})
```

:::tip
While Prisma includes an experimental feature in the migration command that is able to generate SQL from a schema, creating tables and columns using the provided SQL is currently recommended instead as SQL schemas automatically generated by Prisma may differ from the recommended schemas.
:::
Schema for the Prisma Adapter (`@next-auth/prisma-adapter`)

## Setup

Create a schema file in `prisma/schema.prisma` similar to this one:

```json title="schema.prisma"
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

model Account {
  id                 String    @id @default(cuid())
  userId             String
  providerType       String
  providerId         String
  providerAccountId  String
  refreshToken       String?
  accessToken        String?
  accessTokenExpires DateTime?
  createdAt          DateTime  @default(now())
  updatedAt          DateTime  @updatedAt
  user               User      @relation(fields: [userId], references: [id])

  @@unique([providerId, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  userId       String
  expires      DateTime
  sessionToken String   @unique
  accessToken  String   @unique
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  user         User     @relation(fields: [userId], references: [id])
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  accounts      Account[]
  sessions      Session[]
}

model VerificationRequest {
  id         String   @id @default(cuid())
  identifier String
  token      String   @unique
  expires    DateTime
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@unique([identifier, token])
}

```

### Generate Client

Once you have saved your schema, use the Prisma CLI to generate the Prisma Client:

```
npx prisma generate
```

To configure your database to use the new schema (i.e. create tables and columns) use the `prisma migrate` command:

```
npx prisma migrate dev
```

To generate a schema in this way with the above example code, you will need to specify your database connection string in the environment variable `DATABASE_URL`. You can do this by setting it in a `.env` file at the root of your project.

As this feature is experimental in Prisma, it is behind a feature flag. You should check your database schema manually after using this option. See the [Prisma documentation](https://www.prisma.io/docs/) for information on how to use `prisma migrate`.

## Schema History

Changes from the original Prisma Adapter

```diff
 model Account {
-  id                 Int       @default(autoincrement()) @id
+  id                 String    @id @default(cuid())
-  compoundId         String    @unique @map(name: "compound_id")
-  userId             Int       @map(name: "user_id")
+  userId             String
+  user               User      @relation(fields: [userId], references: [id])
-  providerType       String    @map(name: "provider_type")
+  providerType       String
-  providerId         String    @map(name: "provider_id")
+  providerId         String
-  providerAccountId  String    @map(name: "provider_account_id")
+  providerAccountId  String
-  refreshToken       String?   @map(name: "refresh_token")
+  refreshToken       String?
-  accessToken        String?   @map(name: "access_token")
+  accessToken        String?
-  accessTokenExpires DateTime? @map(name: "access_token_expires")
+  accessTokenExpires DateTime?
-  createdAt          DateTime  @default(now()) @map(name: "created_at")
+  createdAt          DateTime  @default(now())
-  updatedAt          DateTime  @default(now()) @map(name: "updated_at")
+  updatedAt          DateTime  @updatedAt

-  @@index([providerAccountId], name: "providerAccountId")
-  @@index([providerId], name: "providerId")
-  @@index([userId], name: "userId")
-  @@map(name: "accounts")
+  @@unique([providerId, providerAccountId])
 }

 model Session {
-  id           Int      @default(autoincrement()) @id
+  id           String   @id @default(cuid())
-  userId       Int      @map(name: "user_id")
+  userId       String
+  user         User     @relation(fields: [userId], references: [id])
   expires      DateTime
-  sessionToken String   @unique @map(name: "session_token")
+  sessionToken String   @unique
-  accessToken  String   @unique @map(name: "access_token")
+  accessToken  String   @unique
-  createdAt    DateTime @default(now()) @map(name: "created_at")
+  createdAt    DateTime @default(now())
-  updatedAt    DateTime @default(now()) @map(name: "updated_at")
+  updatedAt    DateTime @updatedAt
-
-  @@map(name: "sessions")
 }

 model User {
-  id            Int       @default(autoincrement()) @id
+  id            String    @id @default(cuid())
   name          String?
   email         String?   @unique
-  emailVerified DateTime? @map(name: "email_verified")
+  emailVerified DateTime?
   image         String?
+  accounts      Account[]
+  sessions      Session[]
-  createdAt     DateTime  @default(now()) @map(name: "created_at")
+  createdAt     DateTime  @default(now())
-  updatedAt     DateTime  @default(now()) @map(name: "updated_at")
+  updatedAt     DateTime  @updatedAt

-  @@map(name: "users")
 }

 model VerificationRequest {
-  id         Int      @default(autoincrement()) @id
+  id         String   @id @default(cuid())
   identifier String
   token      String   @unique
   expires    DateTime
-  createdAt  DateTime  @default(now()) @map(name: "created_at")
+  createdAt  DateTime @default(now())
-  updatedAt  DateTime  @default(now()) @map(name: "updated_at")
+  updatedAt  DateTime @updatedAt

-  @@map(name: "verification_requests")
+  @@unique([identifier, token])
 }
```