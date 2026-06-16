## Prisma-Schema-One-to-Many-Relation


```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id String @id @default(cuid())
  email String @unique
  password String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  profile Profile? @relation("UserProfile")
  posts Post[]
}

model Profile {
  id String @id @default(cuid())
  firstName String
  lastName String
  city String
  postalCode String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  userId String @unique
  user User @relation("UserProfile", fields: [userId], references: [id], onDelete: Cascade, onUpdate: Cascade)
}

model Post {
  id String @id @default(cuid())
  title String
  content String
  published Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  authorId String
  author User @relation(fields: [authorId], references: [id], onDelete: Cascade, onUpdate: Cascade)
}
```
---
