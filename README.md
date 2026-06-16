# 🗄️ Prisma — One-to-Many Relation

> **লক্ষ্য:** এই ডকুমেন্টে শিখবো কিভাবে Prisma Schema-তে **One-to-Many** সম্পর্ক তৈরি হয়।  
> ব্যবহৃত উদাহরণ: `User → Posts` (একজন User অনেকগুলো Post লিখতে পারে)

---

## 📌 সম্পর্কের ধরন — এক নজরে

| Relation Type | উদাহরণ | মানে |
|---|---|---|
| **One-to-One** | User ↔ Profile | একজন User-এর একটিই Profile থাকে |
| **One-to-Many** | User → Posts | একজন User অনেকগুলো Post লিখতে পারে |
| **Many-to-Many** | Post ↔ Tags | একটি Post-এ অনেক Tag, একটি Tag অনেক Post-এ থাকতে পারে |

> এই ডকুমেন্টে আমরা **One-to-Many** নিয়ে কাজ করবো।

---

## 🏗️ পূর্ণ Schema

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

// ─────────────────────────────────────────
// Model: User
// ─────────────────────────────────────────
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  profile   Profile? @relation("UserProfile")   // One-to-One (optional)
  posts     Post[]                               // One-to-Many ← এটাই আমাদের focus
}

// ─────────────────────────────────────────
// Model: Profile
// ─────────────────────────────────────────
model Profile {
  id         String   @id @default(cuid())
  firstName  String
  lastName   String
  city       String
  postalCode String
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  userId String @unique
  user   User   @relation("UserProfile", fields: [userId], references: [id], onDelete: Cascade, onUpdate: Cascade)
}

// ─────────────────────────────────────────
// Model: Post
// ─────────────────────────────────────────
model Post {
  id        String   @id @default(cuid())
  title     String
  content   String
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  authorId String
  author   User   @relation(fields: [authorId], references: [id], onDelete: Cascade, onUpdate: Cascade)
}
```

---

## 🔍 One-to-Many — লাইন বাই লাইন ব্যাখ্যা

### 📦 `User` Model-এর দিক থেকে

```prisma
posts Post[]
```

| অংশ | মানে |
|---|---|
| `posts` | এটা একটা virtual field (Database-এ কোনো column নেই) |
| `Post[]` | Array মানে — একজন User-এর **অনেকগুলো** Post থাকতে পারে |

> 💡 **"One"** দিকে থাকে — তাই এখানে `[]` (array) লেখা হয়।

---

### 📦 `Post` Model-এর দিক থেকে

```prisma
authorId String
author   User @relation(fields: [authorId], references: [id], onDelete: Cascade, onUpdate: Cascade)
```

| অংশ | মানে |
|---|---|
| `authorId String` | এটা **Foreign Key** — Database-এ এই column আসলেই তৈরি হবে |
| `author User` | এটা virtual relation field — কোনো column নয় |
| `fields: [authorId]` | Post table-এ কোন column দিয়ে যুক্ত করবো? → `authorId` |
| `references: [id]` | User table-এ কোন column-এর সাথে মেলাবো? → `id` |
| `onDelete: Cascade` | User delete হলে তার সব Post-ও delete হবে |
| `onUpdate: Cascade` | User-এর `id` পরিবর্তন হলে Post-এর `authorId`-ও আপডেট হবে |

> 💡 **"Many"** দিকে থাকে Foreign Key (`authorId`) — এটাই নিয়ম।

---

## 🗺️ Relation Diagram

```
┌──────────────────────┐          ┌──────────────────────────┐
│        User          │          │          Post            │
├──────────────────────┤          ├──────────────────────────┤
│ id (PK)              │◄────┐    │ id (PK)                  │
│ email                │     │    │ title                    │
│ password             │     │    │ content                  │
│ createdAt            │     │    │ published                │
│ updatedAt            │     │    │ createdAt                │
│                      │     │    │ updatedAt                │
│ profile → Profile    │     └────│ authorId (FK)            │
│ posts   → Post[]     │          │ author → User            │
└──────────────────────┘          └──────────────────────────┘

         1                                   N
      (একজন)                           (অনেকগুলো)
```

**পড়ার নিয়ম:** একজন `User` → অনেকগুলো `Post` লিখতে পারে।  
কিন্তু একটি `Post`-এর শুধুমাত্র **একজন** `author` থাকে।

---

## ⚖️ One-to-One vs One-to-Many — পার্থক্য

| বিষয় | One-to-One (User ↔ Profile) | One-to-Many (User → Posts) |
|---|---|---|
| User Model-এ | `profile Profile?` | `posts Post[]` |
| Child Model-এ FK | `userId String @unique` | `authorId String` (unique নেই) |
| `@unique` লাগে? | ✅ হ্যাঁ (একটাই হবে) | ❌ না (অনেক হতে পারে) |
| Relation নাম | `@relation("UserProfile")` | নাম optional |
| Array ব্যবহার | ❌ না | ✅ হ্যাঁ (`Post[]`) |

> 🔑 **মূল পার্থক্য:** One-to-One-এ FK-তে `@unique` থাকে, One-to-Many-তে থাকে না।

---

## 🔗 `onDelete` ও `onUpdate` — Referential Actions

```prisma
@relation(fields: [authorId], references: [id], onDelete: Cascade, onUpdate: Cascade)
```

| Action | কখন কাজ করে | `Cascade` মানে |
|---|---|---|
| `onDelete: Cascade` | Parent (User) delete হলে | Child (Post)-গুলোও delete হবে |
| `onUpdate: Cascade` | Parent-এর PK পরিবর্তন হলে | Child-এর FK-ও আপডেট হবে |

### অন্যান্য অপশন:

| Option | মানে |
|---|---|
| `Cascade` | Parent-এর সাথে Child-ও পরিবর্তন হবে |
| `Restrict` | Parent delete/update করতে দেবে না যদি Child থাকে |
| `SetNull` | Parent delete হলে Child-এর FK `null` হয়ে যাবে |
| `SetDefault` | Parent delete হলে Child-এর FK default value পাবে |
| `NoAction` | কোনো action নেবে না (Database-এর default) |

---

## 🛠️ Migration চালানোর পর Database-এ কী হয়?

```sql
-- User table
CREATE TABLE "User" (
  "id"        TEXT PRIMARY KEY,
  "email"     TEXT UNIQUE NOT NULL,
  "password"  TEXT NOT NULL,
  "createdAt" TIMESTAMP NOT NULL DEFAULT NOW(),
  "updatedAt" TIMESTAMP NOT NULL
);

-- Post table (authorId হলো Foreign Key)
CREATE TABLE "Post" (
  "id"        TEXT PRIMARY KEY,
  "title"     TEXT NOT NULL,
  "content"   TEXT NOT NULL,
  "published" BOOLEAN NOT NULL DEFAULT FALSE,
  "createdAt" TIMESTAMP NOT NULL DEFAULT NOW(),
  "updatedAt" TIMESTAMP NOT NULL,
  "authorId"  TEXT NOT NULL,

  FOREIGN KEY ("authorId") REFERENCES "User"("id")
    ON DELETE CASCADE
    ON UPDATE CASCADE
);
```

> 📌 **লক্ষ্য করো:** `posts Post[]` এবং `author User` — এই virtual field গুলো Database-এ কোনো column তৈরি করে না। শুধু `authorId` column তৈরি হয়।

---

## 🧪 Prisma Client দিয়ে Data Query

### ১. User তৈরি করা (Post সহ)

```typescript
const user = await prisma.user.create({
  data: {
    email: "wasim@example.com",
    password: "hashed_password",
    posts: {
      create: [
        { title: "প্রথম পোস্ট", content: "এটা আমার প্রথম পোস্ট।" },
        { title: "দ্বিতীয় পোস্ট", content: "এটা আমার দ্বিতীয় পোস্ট।" },
      ],
    },
  },
});
```

---

### ২. User-এর সব Post নিয়ে আসা (include)

```typescript
const userWithPosts = await prisma.user.findUnique({
  where: { id: "user_id_here" },
  include: {
    posts: true,  // সব Post নিয়ে আসবে
  },
});

// Result:
// {
//   id: "...",
//   email: "wasim@example.com",
//   posts: [
//     { id: "...", title: "প্রথম পোস্ট", ... },
//     { id: "...", title: "দ্বিতীয় পোস্ট", ... },
//   ]
// }
```

---

### ৩. Post-এর সাথে Author নিয়ে আসা

```typescript
const post = await prisma.post.findUnique({
  where: { id: "post_id_here" },
  include: {
    author: true,  // সেই Post-এর User নিয়ে আসবে
  },
});
```

---

### ৪. নতুন Post যোগ করা (existing User-এ)

```typescript
const newPost = await prisma.post.create({
  data: {
    title: "তৃতীয় পোস্ট",
    content: "নতুন আরেকটি পোস্ট।",
    authorId: "existing_user_id",  // FK সরাসরি দাও
  },
});
```

---

## 📝 মনে রাখার নিয়ম (Summary)

```
✅ One-to-Many Rule:
   "Many" দিকে (Post) → Foreign Key রাখো (authorId)
   "One" দিকে (User)  → Array রাখো (posts Post[])

✅ Foreign Key এর @unique নেই One-to-Many তে
   কারণ একই User-এর অনেক Post থাকতে পারে

✅ Virtual fields (posts, author) Database-এ column তৈরি করে না
   শুধু Prisma Client query-তে কাজে লাগে

✅ onDelete: Cascade → Parent মুছলে Child-ও মুছবে
```

---

## 🔗 এই Schema-তে সব Relation একসাথে

```
User (1) ──────────── (1) Profile    ← One-to-One
User (1) ──────────── (N) Post       ← One-to-Many  ✅ (এই ডকুমেন্ট)
```

> **পরবর্তী:** Many-to-Many Relation (যেমন Post ↔ Tag) শিখতে হবে।

---
