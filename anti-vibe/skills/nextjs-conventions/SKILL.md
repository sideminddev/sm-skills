---
name: nextjs-conventions
description: Frontend/backend conventions for Next.js 14+ App Router projects: Server Components, Client Components, Route Handlers, data fetching, caching, project structure. Use when implementing or reviewing Next.js full-stack code.
---

# Next.js 14+ Conventions

When working in a Next.js 14+ project with App Router, follow these conventions for consistent, maintainable full-stack code.

## Project Structure

```
app/
├── (groups)/           # Route groups for layout organization
│   ├── (marketing)/
│   ├── (dashboard)/
│   └── (auth)/
├── api/                # Route Handlers
├── layout.tsx          # Root layout
├── page.tsx            # Home page
├── loading.tsx         # Global loading UI
├── error.tsx           # Global error UI
├── not-found.tsx       # 404 page
components/
├── ui/                 # Presentational components
├── forms/              # Form-specific components
├── providers/          # Context providers (Client Components)
lib/
├── db.ts               # Database/ORM client
├── auth.ts             # Auth configuration
├── utils.ts            # Utility functions
└── validations/        # Zod/yup schemas
```

## Server Components (Default)

Server Components are the default in the App Router. Use them for:

- Data fetching (ORM, APIs, databases)
- Accessing backend resources directly
- Keeping sensitive data/server logic on the server
- Large dependencies that shouldn't reach the client

```tsx
// app/page.tsx - Server Component by default
import { prisma } from '@/lib/db'

export default async function Dashboard() {
  // Fetch data directly on the server
  const data = await prisma.projects.findMany()
  
  return (
    <ul>
      {data.map((project) => (
        <li key={project.id}>{project.name}</li>
      ))}
    </ul>
  )
}
```

### Data Fetching Patterns

```tsx
// Pattern 1: Direct ORM/database access
async function getData() {
  return await prisma.project.findMany()
}

// Pattern 2: Internal API call (if needed for cross-cutting concerns)
async function getData() {
  const res = await fetch('http://localhost:3000/api/projects', {
    cache: 'no-store', // or 'force-cache' for static data
  })
  return res.json()
}

// Pattern 3: External API with revalidation
async function getData() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 } // Revalidate every hour
  })
  return res.json()
}
```

## Client Components

Mark with `'use client'` when you need:

- Browser APIs (localStorage, window, document)
- React hooks (useState, useEffect, useContext)
- Event handlers (onClick, onSubmit)
- Third-party client libraries

```tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

### Best Practice: Push Client Components Down

Keep as much code as possible on the server. Extract client interactivity into small, focused components:

```tsx
// app/page.tsx - Server Component
import { DataTable } from '@/components/data-table' // Server
import { EditButton } from '@/components/edit-button' // Client

export default async function Page() {
  const data = await fetchData() // Server-side fetch
  
  return (
    <div>
      <DataTable data={data} />
      <EditButton /> {/* Only this part is client-side */}
    </div>
  )
}
```

## Route Handlers (API Routes)

Define in `route.ts` files inside `app/api/` or anywhere in `app/`:

```tsx
// app/api/projects/route.ts
import { NextResponse } from 'next/server'
import { z } from 'zod'

const projectSchema = z.object({
  name: z.string().min(1),
  description: z.string().optional(),
})

// GET /api/projects
export async function GET() {
  try {
    const projects = await prisma.project.findMany()
    return NextResponse.json(projects)
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch projects' },
      { status: 500 }
    )
  }
}

// POST /api/projects
export async function POST(request: Request) {
  try {
    const body = await request.json()
    const validated = projectSchema.parse(body)
    
    const project = await prisma.project.create({
      data: validated,
    })
    
    return NextResponse.json(project, { status: 201 })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      )
    }
    return NextResponse.json(
      { error: 'Failed to create project' },
      { status: 500 }
    )
  }
}
```

### Route Handler Conventions

- Use `NextResponse` for consistent response formatting
- Validate input with Zod or similar before processing
- Return appropriate HTTP status codes (200, 201, 400, 401, 404, 500)
- Handle errors gracefully with try/catch
- **Never** expose sensitive error details to the client

## Route Groups

Use parentheses for route groups that don't affect URL structure:

```
app/
├── (marketing)/
│   ├── layout.tsx      # Marketing layout
│   ├── page.tsx        # / (home)
│   └── about/
│       └── page.tsx    # /about
├── (dashboard)/
│   ├── layout.tsx      # Dashboard layout (sidebar, etc.)
│   ├── dashboard/
│   │   └── page.tsx    # /dashboard
│   └── settings/
│       └── page.tsx    # /dashboard/settings
```

## Loading and Error UI

```tsx
// app/loading.tsx
export default function Loading() {
  return <div className="loading-spinner">Loading...</div>
}

// app/error.tsx
'use client'

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={reset}>Try again</button>
    </div>
  )
}

// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>404 - Page Not Found</h2>
      <p>The requested resource doesn't exist</p>
    </div>
  )
}
```

### Error Boundary Pattern

```tsx
// components/error-boundary.tsx
'use client'

import { Component, ReactNode } from 'react'

interface Props {
  children: ReactNode
  fallback: ReactNode
}

interface State {
  hasError: boolean
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(): State {
    return { hasError: true }
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback
    }
    return this.props.children
  }
}
```

## Caching and Revalidation

### Static Data (Cached by Default)

```tsx
// Cached indefinitely (good for static content)
const data = await fetch('https://api.example.com/static-data')
```

### Dynamic Data (No Cache)

```tsx
// Never cached - always fetch fresh
const data = await fetch('https://api.example.com/live-data', {
  cache: 'no-store',
})
```

### Time-based Revalidation

```tsx
// Revalidate every hour
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 },
})
```

### On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache'

export async function POST(request: Request) {
  const { path } = await request.json()
  revalidatePath(path)
  return NextResponse.json({ revalidated: true })
}
```

## Middleware

```tsx
// middleware.ts (in project root)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Authentication check
  const token = request.cookies.get('token')
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/protected/:path*'],
}
```

## Server Actions (Optional Pattern)

For forms and mutations, Server Actions provide a way to call server functions directly from components:

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createProject(formData: FormData) {
  const name = formData.get('name') as string
  
  await prisma.project.create({
    data: { name },
  })
  
  revalidatePath('/dashboard')
}

// app/page.tsx
import { createProject } from './actions'

export default function Page() {
  return (
    <form action={createProject}>
      <input name="name" required />
      <button type="submit">Create</button>
    </form>
  )
}
```

## Conventions Summary

| Pattern | Recommendation |
|---------|----------------|
| **Default component type** | Server Component (no directive needed) |
| **Client interactivity** | Extract to small `'use client'` components |
| **Data fetching** | Server Components → direct ORM/API calls |
| **Form submissions** | Server Actions or Route Handlers with validation |
| **API routes** | Route Handlers in `app/api/.../route.ts` |
| **Layout organization** | Route groups `(groupname)/` for shared layouts |
| **Error handling** | `error.tsx` files + Error Boundaries for client parts |
| **Loading states** | `loading.tsx` files for automatic Suspense boundaries |
| **Validation** | Zod for runtime validation everywhere |
| **Caching** | Use `cache: 'no-store'` for dynamic, `revalidate` for semi-static |

## TypeScript Configuration

Ensure `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

## What to Check

1. **Server/Client Boundary**: Is `'use client'` only used where truly needed?
2. **Data Fetching**: Are Server Components fetching directly when possible?
3. **Route Handlers**: Proper HTTP methods, input validation, error handling?
4. **Error Boundaries**: Are errors caught and handled gracefully?
5. **Caching Strategy**: Appropriate cache settings for data freshness needs?
6. **Type Safety**: Proper TypeScript types throughout, no `any` without justification?

Reference project rules and Next.js official docs for framework-specific patterns.
