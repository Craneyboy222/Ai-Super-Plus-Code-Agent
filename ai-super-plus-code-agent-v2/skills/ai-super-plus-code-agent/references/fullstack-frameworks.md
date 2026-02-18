# Fullstack Frameworks — Production Configurations

## Frontend Frameworks

### Next.js 14 (Full Stack)

**Setup**:
```bash
npm create next-app@latest --typescript --tailwind --eslint
cd project && npm install
```

**Key Files**:
```
app/
├── layout.tsx          # Root layout
├── page.tsx            # Home page
├── api/
│   ├── auth/[...nextauth]/route.ts
│   └── posts/route.ts
├── components/
│   ├── Header.tsx
│   ├── Navigation.tsx
│   └── ui/
│       ├── Button.tsx
│       ├── Input.tsx
│       └── Card.tsx
├── hooks/
│   ├── useAuth.ts
│   └── usePost.ts
├── lib/
│   ├── api-client.ts
│   └── auth.ts
└── types/
    └── index.ts

next.config.js
tailwind.config.ts
tsconfig.json
```

**API Route Example**:
```typescript
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { db } from '@/lib/db';
import { CreatePostSchema } from '@/lib/schemas';

export async function POST(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await req.json();
    const data = CreatePostSchema.parse(body);

    const post = await db.post.create({
      data: {
        ...data,
        userId: session.user.id
      }
    });

    return NextResponse.json(post, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Internal error' },
      { status: 500 }
    );
  }
}

export async function GET(req: NextRequest) {
  try {
    const { searchParams } = new URL(req.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '20');

    const posts = await db.post.findMany({
      skip: (page - 1) * limit,
      take: limit,
      include: { author: true, _count: { select: { comments: true } } }
    });

    return NextResponse.json(posts);
  } catch (error) {
    return NextResponse.json({ error: 'Failed to fetch posts' }, { status: 500 });
  }
}
```

**Server Component Example**:
```typescript
// app/posts/page.tsx
import { Metadata } from 'next';
import { PostCard } from '@/components/PostCard';
import { db } from '@/lib/db';

export const metadata: Metadata = {
  title: 'Posts',
  description: 'Browse all blog posts'
};

export default async function PostsPage() {
  const posts = await db.post.findMany({
    where: { published: true },
    include: { author: true },
    orderBy: { createdAt: 'desc' }
  });

  return (
    <div className="max-w-4xl mx-auto py-8">
      <h1 className="text-4xl font-bold mb-8">Posts</h1>
      <div className="grid gap-6">
        {posts.map(post => (
          <PostCard key={post.id} post={post} />
        ))}
      </div>
    </div>
  );
}
```

---

### React 18 + Vite

**Setup**:
```bash
npm create vite@latest my-app -- --template react-ts
cd my-app && npm install
npm install -D @testing-library/react @testing-library/jest-dom vitest
```

**Structure**:
```
src/
├── components/
│   ├── PostList.tsx
│   ├── PostForm.tsx
│   └── common/
│       ├── Button.tsx
│       └── Input.tsx
├── pages/
│   ├── HomePage.tsx
│   ├── PostPage.tsx
│   └── LoginPage.tsx
├── hooks/
│   ├── usePost.ts
│   └── useAuth.ts
├── services/
│   └── api.ts
├── store/
│   └── authStore.ts
├── App.tsx
└── main.tsx
```

**Store (Zustand)**:
```typescript
// src/store/authStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  role: 'user' | 'admin';
}

interface AuthStore {
  user: User | null;
  token: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

export const useAuthStore = create<AuthStore>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        token: null,
        isAuthenticated: false,
        login: async (email: string, password: string) => {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            body: JSON.stringify({ email, password })
          });
          const data = await response.json();
          set({ user: data.user, token: data.token, isAuthenticated: true });
        },
        logout: () => set({ user: null, token: null, isAuthenticated: false })
      }),
      { name: 'auth-storage' }
    )
  )
);
```

**Component with Hooks**:
```typescript
// src/components/PostForm.tsx
import { useState } from 'react';
import { useMutation } from '@tanstack/react-query';
import { createPost } from '@/services/api';
import { useAuthStore } from '@/store/authStore';

export const PostForm = () => {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const user = useAuthStore(s => s.user);

  const mutation = useMutation({
    mutationFn: () => createPost({ title, content }),
    onSuccess: () => {
      setTitle('');
      setContent('');
    }
  });

  if (!user) return <p>Please log in</p>;

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      mutation.mutate();
    }}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
      />
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Content"
      />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create'}
      </button>
      {mutation.isError && <p>Error: {(mutation.error as Error).message}</p>}
    </form>
  );
};
```

---

## Backend Frameworks

### Express.js + TypeScript

**Setup**:
```bash
npm init -y
npm install express cors dotenv zod @prisma/client bcrypt jsonwebtoken
npm install -D typescript @types/express @types/node tsx jest @types/jest ts-jest
```

**Server Setup**:
```typescript
// src/server.ts
import express, { Express, Request, Response, NextFunction } from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { errorHandler } from './middleware/errorHandler';
import { requestLogger } from './middleware/requestLogger';
import { routes } from './routes';

const app: Express = express();

// Security middleware
app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') }));

// Parsing middleware
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ limit: '10mb', extended: true }));

// Logging
app.use(requestLogger);

// Health check
app.get('/health', (req: Request, res: Response) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// API routes
app.use('/api', routes);

// Error handling
app.use(errorHandler);

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Route Handler**:
```typescript
// src/routes/posts.ts
import { Router, Request, Response, NextFunction } from 'express';
import { authenticate } from '@/middleware/auth';
import { validateInput } from '@/middleware/validation';
import { postController } from '@/controllers/postController';
import { CreatePostSchema } from '@/schemas';

export const postRouter = Router();

// Create post
postRouter.post(
  '/',
  authenticate,
  validateInput(CreatePostSchema),
  postController.create
);

// Get post
postRouter.get('/:id', postController.getOne);

// List posts
postRouter.get('/', postController.list);

// Update post
postRouter.patch(
  '/:id',
  authenticate,
  validateInput(CreatePostSchema.partial()),
  postController.update
);

// Delete post
postRouter.delete('/:id', authenticate, postController.delete);
```

**Controller**:
```typescript
// src/controllers/postController.ts
import { Request, Response } from 'express';
import { AuthRequest } from '@/types';
import { postService } from '@/services/postService';

export const postController = {
  create: async (req: AuthRequest, res: Response) => {
    try {
      const post = await postService.create({
        ...req.body,
        userId: req.user!.id
      });
      res.status(201).json(post);
    } catch (error) {
      res.status(500).json({ error: 'Failed to create post' });
    }
  },

  getOne: async (req: Request, res: Response) => {
    try {
      const post = await postService.getById(req.params.id);
      if (!post) return res.status(404).json({ error: 'Not found' });
      res.json(post);
    } catch (error) {
      res.status(500).json({ error: 'Failed to fetch post' });
    }
  },

  list: async (req: Request, res: Response) => {
    try {
      const page = parseInt(req.query.page as string) || 1;
      const limit = parseInt(req.query.limit as string) || 20;

      const posts = await postService.list({ page, limit });
      res.json(posts);
    } catch (error) {
      res.status(500).json({ error: 'Failed to fetch posts' });
    }
  },

  update: async (req: AuthRequest, res: Response) => {
    try {
      const post = await postService.update(req.params.id, req.body, req.user!.id);
      res.json(post);
    } catch (error) {
      res.status(500).json({ error: 'Failed to update post' });
    }
  },

  delete: async (req: AuthRequest, res: Response) => {
    try {
      await postService.delete(req.params.id, req.user!.id);
      res.status(204).send();
    } catch (error) {
      res.status(500).json({ error: 'Failed to delete post' });
    }
  }
};
```

---

### FastAPI (Python)

**Setup**:
```bash
pip install fastapi uvicorn pydantic sqlalchemy asyncpg python-jose[cryptography]
pip install pytest pytest-asyncio httpx
```

**Application**:
```python
# app/main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.ext.asyncio import AsyncSession
from contextlib import asynccontextmanager

from app.db import engine, get_db
from app.models import Base
from app.api import routers

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: Create tables
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    # Shutdown
    await engine.dispose()

app = FastAPI(
    title="Blog API",
    description="Production blog API",
    version="1.0.0",
    lifespan=lifespan
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Routes
app.include_router(routers.posts.router, prefix="/api/posts", tags=["posts"])
app.include_router(routers.auth.router, prefix="/api/auth", tags=["auth"])

@app.get("/health")
async def health():
    return {"status": "ok"}
```

**Model and Schema**:
```python
# app/models.py
from sqlalchemy import Column, String, Text, DateTime, ForeignKey, Boolean
from sqlalchemy.orm import declarative_base, relationship
from datetime import datetime
from uuid import uuid4

Base = declarative_base()

class Post(Base):
    __tablename__ = "posts"

    id = Column(String, primary_key=True, default=lambda: str(uuid4()))
    title = Column(String(255), nullable=False, index=True)
    content = Column(Text, nullable=False)
    user_id = Column(String, ForeignKey("users.id"), nullable=False)
    published = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    user = relationship("User", back_populates="posts")

# app/schemas.py
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional

class PostCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=255)
    content: str = Field(..., min_length=1)
    published: bool = False

class PostResponse(PostCreate):
    id: str
    user_id: str
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True
```

**Route Handler**:
```python
# app/api/routers/posts.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

from app.db import get_db
from app.models import Post
from app.schemas import PostCreate, PostResponse
from app.auth import get_current_user

router = APIRouter()

@router.post("", response_model=PostResponse, status_code=status.HTTP_201_CREATED)
async def create_post(
    post_data: PostCreate,
    db: AsyncSession = Depends(get_db),
    current_user = Depends(get_current_user)
):
    db_post = Post(
        title=post_data.title,
        content=post_data.content,
        published=post_data.published,
        user_id=current_user.id
    )
    db.add(db_post)
    await db.commit()
    await db.refresh(db_post)
    return db_post

@router.get("/{post_id}", response_model=PostResponse)
async def get_post(post_id: str, db: AsyncSession = Depends(get_db)):
    stmt = select(Post).where(Post.id == post_id)
    result = await db.execute(stmt)
    post = result.scalar_one_or_none()

    if not post:
        raise HTTPException(status_code=404, detail="Post not found")

    return post
```

---

## Database

### Prisma Schema

```prisma
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  posts     Post[]
  comments  Comment[]
  teams     Team[]   @relation("TeamMembers", through: TeamMember)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String
  published Boolean  @default(false)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  comments  Comment[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([userId])
  @@index([published])
}

model Comment {
  id        String   @id @default(cuid())
  content   String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([userId])
}

model Team {
  id        String   @id @default(cuid())
  name      String
  members   TeamMember[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model TeamMember {
  team      Team   @relation(fields: [teamId], references: [id], onDelete: Cascade)
  teamId    String
  user      User   @relation("TeamMembers", fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  role      String @default("member")

  @@id([teamId, userId])
  @@index([teamId])
  @@index([userId])
}

enum Role {
  USER
  ADMIN
}
```

---

## Complete Example: Blog API Endpoint

**Request/Response Flow**:
```
POST /api/posts
├── Body: { title: "My Post", content: "..." }
├── Auth Header: "Authorization: Bearer TOKEN"
└── Response: { id: "123", title: "...", userId: "...", createdAt: "..." }

GET /api/posts?page=1&limit=20
├── Query Parameters
└── Response: [ { id: "123", title: "..." }, ... ]

GET /api/posts/123
└── Response: { id: "123", title: "...", comments: [...] }

PATCH /api/posts/123
├── Body: { title: "Updated Title" }
└── Response: { id: "123", title: "Updated Title", ... }

DELETE /api/posts/123
└── Response: 204 No Content
```

All frameworks configured for:
- Type safety
- Error handling
- Input validation
- Authentication
- Pagination
- Filtering
- Rate limiting
