## 🧩 **Phase 1 — Base Setup**

---

### **1. Objective**

Establish the foundational environment and infrastructure for the **Chef Digital** web application:

* Initialize a **Next.js 14 (App Router)** full-stack project.
* Configure **TypeScript**, **TailwindCSS**, and **shadcn/ui**.
* Connect **Prisma ORM** to a **PostgreSQL** database (Neon or Supabase).
* Set up **Google OAuth login** using **Auth.js (NextAuth)** restricted to your personal email.
* Prepare secure environment variables and deploy-ready configuration for **Render**.

---

### **2. Architecture Decisions**

| Component      | Technology                     | Reasoning                                                     |
| -------------- | ------------------------------ | ------------------------------------------------------------- |
| Framework      | **Next.js 14 (App Router)**    | Full-stack with built-in API routes, best practice in 2025    |
| Styling        | **TailwindCSS + shadcn/ui**    | Modern, accessible, consistent design system                  |
| Database       | **PostgreSQL (Neon/Supabase)** | Reliable relational DB with free tiers and good integrations  |
| ORM            | **Prisma**                     | Type-safe queries, migrations, schema-first approach          |
| Authentication | **Auth.js (NextAuth)**         | Plug-and-play OAuth with session handling                     |
| Deployment     | **Render**                     | Simple build flow, environment variable management, free tier |

---

### **3. Implementation Steps**

#### **Step 1 — Create the Next.js project**

```bash
npx create-next-app@latest chef-digital --typescript --app
cd chef-digital
```

When prompted:

* ✅ Use TypeScript
* ✅ Use ESLint
* ✅ Use TailwindCSS
* ✅ Use App Router
* ✅ Configure alias `@/*`

---

#### **Step 2 — Install dependencies**

```bash
npm install @prisma/client prisma next-auth @auth/prisma-adapter
npm install lucide-react clsx zod axios
npm install -D tailwind-merge
```

---

#### **Step 3 — Configure TailwindCSS**

Ensure `tailwind.config.ts` contains:

```typescript
import { fontFamily } from "tailwindcss/defaultTheme";

export default {
  content: [
    "./app/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./pages/**/*.{ts,tsx}",
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ["Inter", ...fontFamily.sans],
      },
    },
  },
  plugins: [],
};
```

---

- [x] Tailwind base configuration implemented (Step 3)

#### **Step 4 — Initialize shadcn/ui**

```bash
npx shadcn@latest init
```

Then install base components:

```bash
npx shadcn add button input card textarea label
```

> _Note:_ The former `shadcn-ui` package has been deprecated; use the maintained `shadcn` CLI shown above.

---

#### **Step 5 — Set up Prisma**

```bash
npx prisma init
```

This creates a `prisma/schema.prisma` file and `.env` file.

**Example schema (for now):**

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

Run migration setup:

```bash
npx prisma migrate dev --name init
```

---

#### **Step 6 — Set up Auth.js (NextAuth)**

Create the file: `app/api/auth/[...nextauth]/route.ts`

```typescript
import NextAuth from "next-auth";
import GoogleProvider from "next-auth/providers/google";
import { PrismaAdapter } from "@auth/prisma-adapter";
import prisma from "@/lib/prisma";

const handler = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async signIn({ profile }) {
      return profile?.email === process.env.ALLOWED_EMAIL;
    },
  },
});

export { handler as GET, handler as POST };
```

In `.env`:

```bash
DATABASE_URL="postgresql://user:password@host:port/db"
GOOGLE_CLIENT_ID="your-client-id"
GOOGLE_CLIENT_SECRET="your-client-secret"
NEXTAUTH_SECRET="your-secret"
ALLOWED_EMAIL="youremail@example.com"
```

---

#### **Step 7 — Create Prisma Client**

`lib/prisma.ts`

```typescript
import { PrismaClient } from "@prisma/client";
const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ["query", "error", "warn"],
  });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;

export default prisma;
```

---

#### **Step 8 — Add a Basic Layout and Test Auth**

Create `app/layout.tsx`:

```tsx
import "./globals.css";
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export const metadata = {
  title: "Chef Digital",
  description: "Your AI-powered recipe assistant",
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={`${inter.className} bg-white text-gray-900`}>
        <main className="max-w-3xl mx-auto p-4">{children}</main>
      </body>
    </html>
  );
}
```

Create a basic test route `app/page.tsx`:

```tsx
export default function Home() {
  return (
    <div className="space-y-4">
      <h1 className="text-3xl font-bold">Chef Digital</h1>
      <p>Welcome to your AI-powered recipe manager!</p>
    </div>
  );
}
```

Run the dev server:

```bash
npm run dev
```

Verify at: [http://localhost:3000](http://localhost:3000)

---

### **4. Verification & Testing**

| Test                | Expected Result                             |
| ------------------- | ------------------------------------------- |
| App builds and runs | ✅ `localhost:3000` displays welcome message |
| Tailwind styles     | ✅ Styles render correctly                   |
| Prisma migration    | ✅ Database connected and tables created     |
| Auth route          | ✅ `/api/auth/signin` prompts Google login   |
| Access restriction  | ✅ Only the configured email can sign in     |

---

### **5. Deliverables after Phase 1**

✅ Next.js + Tailwind + shadcn/ui configured
✅ Prisma connected to PostgreSQL
✅ Auth.js (Google OAuth) working
✅ Environment variables secured
✅ Basic UI rendering
