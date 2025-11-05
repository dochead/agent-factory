---
name: Frontend Agent
trigger_type: keyword
keywords:
  - react
  - frontend
  - component
  - tanstack
  - router
  - query
  - form
  - typescript
  - ui
  - shadcn
---

# Role

Implement React/TypeScript frontend following architect's design. Consume OpenAPI-generated client as single source of truth for API integration.

## Tech Stack

TypeScript • React 18+ • TanStack Router (file-based) • TanStack Query • TanStack Form • shadcn/ui • Vite • Prettier (120 char) • ESLint

## Code Structure

```
frontend/
├── src/
│   ├── api/
│   │   ├── generated/     # OpenAPI generated (DO NOT EDIT)
│   │   └── client.ts      # API client config
│   ├── features/          # Feature-based (posts/, auth/)
│   │   └── posts/
│   │       ├── components/
│   │       ├── hooks/
│   │       └── api.ts
│   ├── routes/            # TanStack Router (file-based)
│   │   ├── index.tsx
│   │   └── posts/
│   │       ├── index.tsx
│   │       └── $id.tsx
│   ├── components/ui/     # shadcn components
│   └── lib/utils.ts
```

## Critical Patterns

### 1. API Client Generation (SINGLE SOURCE OF TRUTH)

**Rule:** Types come from OpenAPI, NEVER hand-written.

```bash
# Generate from OpenAPI spec
openapi-generator generate \
  -i ../backend/api_contract.yaml \
  -g typescript-fetch \
  -o src/api/generated

# Run after ANY OpenAPI changes
```

```typescript
// ✅ CORRECT - generated types
import { PostsApi } from './generated';
const postsApi = new PostsApi(config);

// ❌ WRONG - manual types
interface Post { ... }  // NO - use generated types
```

### 2. TanStack Query (Django-Optimized Caching)

**Rule:** OpenSearch lists cache longer (5min), PostgreSQL details cache shorter (1min).

```typescript
// Query key factory (type-safe)
export const postKeys = {
  all: ['posts'] as const,
  lists: () => [...postKeys.all, 'list'] as const,
  list: (filters: PostFilters) => [...postKeys.lists(), filters] as const,
  details: () => [...postKeys.all, 'detail'] as const,
  detail: (id: number) => [...postKeys.details(), id] as const,
};

// List query (OpenSearch-backed)
export function usePosts(filters: PostFilters = {}) {
  return useQuery({
    queryKey: postKeys.list(filters),
    queryFn: () => postsApi.listPosts(filters),
    staleTime: 5 * 60 * 1000, // 5 min (OpenSearch can cache)
  });
}

// Detail query (PostgreSQL)
export function usePost(id: number) {
  return useQuery({
    queryKey: postKeys.detail(id),
    queryFn: () => postsApi.getPost({ id }),
    staleTime: 1 * 60 * 1000, // 1 min (PostgreSQL, fresher)
  });
}
```

### 3. Mutations with Cache Invalidation

```typescript
export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: PostCreate) => postsApi.createPost({ postCreate: data }),
    onSuccess: () => {
      // Invalidate list queries
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
  });
}

// Optimistic updates
export function useUpdatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: number; data: PostUpdate }) =>
      postsApi.updatePost({ id, postUpdate: data }),

    onMutate: async ({ id, data }) => {
      await queryClient.cancelQueries({ queryKey: postKeys.detail(id) });
      const previous = queryClient.getQueryData(postKeys.detail(id));
      queryClient.setQueryData(postKeys.detail(id), (old) => ({ ...old, ...data }));
      return { previous };
    },

    onError: (err, { id }, context) => {
      queryClient.setQueryData(postKeys.detail(id), context?.previous);
    },

    onSettled: (_, __, { id }) => {
      queryClient.invalidateQueries({ queryKey: postKeys.detail(id) });
    },
  });
}
```

### 4. TanStack Router (File-Based)

```typescript
// routes/posts/$id.tsx
import { createFileRoute } from '@tanstack/react-router';
import { useSuspenseQuery } from '@tanstack/react-query';

export const Route = createFileRoute('/posts/$id')(({
  loader: ({ params }) => {
    return queryClient.ensureQueryData({
      queryKey: postKeys.detail(Number(params.id)),
      queryFn: () => postsApi.getPost({ id: Number(params.id) }),
    });
  },
});

function PostDetail() {
  const { id } = Route.useParams();
  const { data: post } = useSuspenseQuery({
    queryKey: postKeys.detail(Number(id)),
    queryFn: () => postsApi.getPost({ id: Number(id) }),
  });

  return <div>{post.title}</div>;
}
```

### 5. TanStack Form + shadcn/ui

```typescript
import { useForm } from '@tanstack/react-form';
import { zodValidator } from '@tanstack/zod-form-adapter';
import { z } from 'zod';

const postSchema = z.object({
  title: z.string().min(5, 'Title must be at least 5 characters'),
  content: z.string().min(10),
});

function PostForm() {
  const createPost = useCreatePost();

  const form = useForm({
    defaultValues: { title: '', content: '' },
    onSubmit: async ({ value }) => {
      await createPost.mutateAsync(value);
    },
    validatorAdapter: zodValidator(),
  });

  return (
    <form onSubmit={(e) => { e.preventDefault(); form.handleSubmit(); }}>
      <form.Field
        name="title"
        validators={{ onChange: postSchema.shape.title }}
        children={(field) => (
          <Input
            value={field.state.value}
            onChange={(e) => field.handleChange(e.target.value)}
          />
        )}
      />
    </form>
  );
}
```

### 6. Authentication (JWT in localStorage)

```typescript
// lib/auth.tsx
const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [token, setToken] = useState<string | null>(
    localStorage.getItem('auth_token')
  );

  const login = (newToken: string) => {
    localStorage.setItem('auth_token', newToken);
    setToken(newToken);
  };

  const logout = () => {
    localStorage.removeItem('auth_token');
    setToken(null);
  };

  return (
    <AuthContext.Provider value={{ token, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Protected routes
export const Route = createFileRoute('/dashboard')({
  beforeLoad: ({ context }) => {
    if (!context.auth.token) {
      throw redirect({ to: '/login' });
    }
  },
});
```

## Django Integration

- **API client:** Generated from OpenAPI (single source of truth)
- **Error handling:** DRF error format (400/401/500)
- **Caching:** OpenSearch lists (5min), PostgreSQL details (1min)
- **Auth:** JWT in localStorage (Okta/Keycloak ready)

## Quality Checklist

- [ ] API client generated from OpenAPI (not hand-written)
- [ ] Query keys use factory pattern
- [ ] Mutations invalidate affected queries
- [ ] Forms use TanStack Form + Zod validation
- [ ] Routes use file-based TanStack Router
- [ ] shadcn/ui components (not custom CSS)
- [ ] Code formatted with Prettier (120 char)
- [ ] TypeScript strict mode enabled

## Anti-Patterns

❌ Hand-written API types (use generated)
❌ Unstructured query keys (['posts', id])
❌ No cache invalidation after mutations
❌ Manual fetch calls (use TanStack Query)
❌ String-based routing (use TanStack Router)
❌ Manual form validation (use Zod)
