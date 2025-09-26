# UI Implementation Guide

A comprehensive guide for implementing the frontend interfaces of the Queue Management System using modern web technologies and white minimalist design principles.

## 🎯 Overview

The system consists of three distinct user interfaces:

1. **Display Interface** - Customer-facing showcase display
2. **Staff Interface** - Queue management for staff
3. **Super Admin Interface** - System administration

## 🛠️ Technology Stack

### Recommended Frontend Technologies

- **Framework**: React 18+ with TypeScript
- **State Management**: Zustand + TanStack Query
- **Data Fetching**: TanStack Query (React Query)
- **Validation**: Zod
- **Styling**: Tailwind CSS + shadcn + radix
- **Real-time**: Socket.io Client
- **Icons**: Heroicons or Lucide React
- **Animations**: Framer Motion

### Dependencies

```json
{
  "react": "^18.0.0",
  "typescript": "^5.0.0",
  "tailwindcss": "^3.0.0",
  "socket.io-client": "^4.0.0",
  "framer-motion": "^10.0.0",
  "react-router-dom": "^6.0.0",
  "axios": "^1.0.0",
  "@headlessui/react": "^1.7.0",
  "@tanstack/react-query": "^5.0.0",
  "@tanstack/react-query-devtools": "^5.0.0",
  "zod": "^3.22.0",
  "zustand": "^4.4.0",
  "react-hook-form": "^7.48.0",
  "@hookform/resolvers": "^3.3.0"
}
```

## 🎨 Design System

### Color Palette

```css
:root {
  /* Primary Colors */
  --color-white: #ffffff;
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-200: #e5e7eb;
  --color-gray-300: #d1d5db;
  --color-gray-400: #9ca3af;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-700: #374151;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;

  /* Accent Colors */
  --color-blue-500: #3b82f6;
  --color-green-500: #10b981;
  --color-red-500: #ef4444;
  --color-amber-500: #f59e0b;
}
```

### Typography Scale

```css
/* Font Sizes */
.text-xs {
  font-size: 0.75rem;
}
.text-sm {
  font-size: 0.875rem;
}
.text-base {
  font-size: 1rem;
}
.text-lg {
  font-size: 1.125rem;
}
.text-xl {
  font-size: 1.25rem;
}
.text-2xl {
  font-size: 1.5rem;
}
.text-3xl {
  font-size: 1.875rem;
}
.text-4xl {
  font-size: 2.25rem;
}
.text-6xl {
  font-size: 3.75rem;
}
.text-8xl {
  font-size: 6rem;
}
```

## 🖥️ Interface Specifications

### 1. Display Interface (Showcase UI)

**Purpose**: Customer-facing display showing queue status
**Target Screen**: Large display monitor (1920x1080 or larger)

#### Layout Structure

```
┌─────────────────────────────────────────┐
│                HEADER                   │
│        [Shop Name/Logo]                 │
├─────────────────────────────────────────┤
│                                         │
│        NOW SERVING                      │
│          [A001]                         │
│                                         │
├─────────────────────────────────────────┤
│    NEXT IN QUEUE        RECENT SERVED   │
│      A002  A003           A000  B005    │
│      B001  C001           C003  A999    │
│                                         │
└─────────────────────────────────────────┘
```

#### Data Schemas & Validation

```typescript
// schemas/tokenSchemas.ts
import { z } from "zod";

export const tokenSchema = z.object({
  id: z.string().uuid(),
  number: z.string().min(1),
  customerType: z.enum(["instant", "browser", "retail"]),
  status: z.enum(["waiting", "called", "serving", "completed", "cancelled"]),
  priority: z.number().min(0).max(10),
  createdAt: z.string().datetime(),
  calledAt: z.string().datetime().optional(),
  servedAt: z.string().datetime().optional(),
  completedAt: z.string().datetime().optional(),
  servedBy: z.string().uuid().optional(),
  notes: z.string().optional(),
});

export const queueStatusSchema = z.object({
  currentServing: tokenSchema.nullable(),
  nextInQueue: z.array(tokenSchema),
  recentlyServed: z.array(tokenSchema),
  stats: z.object({
    totalWaiting: z.number(),
    averageWaitTime: z.number(),
    totalServedToday: z.number(),
  }),
});

export type Token = z.infer<typeof tokenSchema>;
export type QueueStatus = z.infer<typeof queueStatusSchema>;
```

#### Component Structure

```typescript
// DisplayInterface.tsx
import { useQuery } from "@tanstack/react-query";
import { queueStatusSchema, type QueueStatus } from "../schemas/tokenSchemas";

interface QueueDisplayProps {
  currentServing: Token | null;
  nextInQueue: Token[];
  recentServed: Token[];
}
```

#### Key Features

- **Real-time Updates**: WebSocket connection for instant updates
- **Large Typography**: Easy readability from distance
- **Status Indicators**: Color-coded token types
- **Animation**: Smooth transitions for token changes

#### TanStack Query Setup

```typescript
// lib/queryClient.ts
import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 10, // 10 minutes
      refetchOnWindowFocus: false,
      retry: (failureCount, error: any) => {
        if (error?.response?.status === 404) return false;
        return failureCount < 3;
      },
    },
    mutations: {
      retry: 1,
    },
  },
});
```

#### Data Fetching Hooks

```typescript
// hooks/useQueueData.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { queueService } from "../services/queueService";
import { queueStatusSchema, tokenSchema } from "../schemas/tokenSchemas";

export const useQueueStatus = () => {
  return useQuery({
    queryKey: ["queue", "status"],
    queryFn: async () => {
      const data = await queueService.getQueueStatus();
      return queueStatusSchema.parse(data);
    },
    refetchInterval: 30000, // Fallback refresh every 30s
  });
};

export const useCreateToken = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: queueService.createToken,
    onSuccess: (newToken) => {
      // Validate the response
      const validatedToken = tokenSchema.parse(newToken);

      // Optimistically update the queue status
      queryClient.setQueryData(["queue", "status"], (old: any) => {
        if (!old) return old;
        return {
          ...old,
          nextInQueue: [...old.nextInQueue, validatedToken].sort((a, b) => {
            if (a.priority !== b.priority) return b.priority - a.priority;
            return (
              new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
            );
          }),
        };
      });

      // Invalidate to refetch fresh data
      queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
    },
  });
};

export const useCallNext = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: queueService.callNext,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
    },
  });
};

export const useCompleteService = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: queueService.completeService,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
    },
  });
};
```

#### Sample Component with TanStack Query

```tsx
const DisplayInterface: React.FC = () => {
  const { data: queueStatus, isLoading, error } = useQueueStatus();

  // Real-time updates still work via WebSocket
  useRealTimeUpdates();

  if (isLoading) {
    return (
      <div className="min-h-screen bg-white flex items-center justify-center">
        <div className="text-2xl font-light text-gray-600">Loading...</div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="min-h-screen bg-white flex items-center justify-center">
        <div className="text-2xl font-light text-red-600">
          Failed to load queue data
        </div>
      </div>
    );
  }

  const { currentServing, nextInQueue, recentlyServed, stats } =
    queueStatus || {};

  return (
    <div className="min-h-screen bg-white p-8">
      {/* Header */}
      <header className="text-center mb-12">
        <h1 className="text-4xl font-light text-gray-800">
          Queue Management System
        </h1>
        <p className="text-gray-600 mt-2">
          Total waiting: {stats?.totalWaiting || 0} | Avg wait:{" "}
          {stats?.averageWaitTime || 0}min
        </p>
      </header>

      {/* Current Serving */}
      <section className="text-center mb-16">
        <h2 className="text-3xl font-light text-gray-600 mb-8">NOW SERVING</h2>
        <div className="text-8xl font-bold text-gray-900">
          {currentServing?.number || "---"}
        </div>
        {currentServing && (
          <div className="mt-4">
            <span
              className={`inline-block px-4 py-2 rounded-full text-white ${
                currentServing.customerType === "instant"
                  ? "bg-blue-500"
                  : currentServing.customerType === "browser"
                  ? "bg-green-500"
                  : "bg-purple-500"
              }`}
            >
              {currentServing.customerType.charAt(0).toUpperCase() +
                currentServing.customerType.slice(1)}
            </span>
          </div>
        )}
      </section>

      {/* Queue Status */}
      <div className="grid grid-cols-2 gap-16">
        <QueueSection title="Next in Queue" tokens={nextInQueue || []} />
        <QueueSection title="Recently Served" tokens={recentlyServed || []} />
      </div>
    </div>
  );
};
```

### 2. Staff Interface

**Purpose**: Queue management for shop staff
**Target Device**: Tablet or desktop

#### Layout Structure

```
┌─────────────────────────────────────────┐
│  [Logo] Queue Management    [User Menu] │
├─────────────────────────────────────────┤
│                                         │
│  Current: A001 [Complete] [Next]        │
│                                         │
│  ┌─ Queue (12) ────────────────────────┐ │
│  │ A002 - Walk-in Instant    [Call]   │ │
│  │ A003 - Walk-in Browser    [Call]   │ │
│  │ B001 - Retail Customer    [Call]   │ │
│  │ B002 - Walk-in Instant    [Call]   │ │
│  └─────────────────────────────────────┘ │
│                                         │
│  [+ New Token]  [Pause Queue]          │
│                                         │
└─────────────────────────────────────────┘
```

#### Key Features

- **Token Management**: Call next, complete service, add new tokens
- **Queue Overview**: See all pending tokens
- **Customer Type Selection**: When creating new tokens
- **Service Actions**: Mark as served, skip, priority handling

#### Form Validation with Zod

```typescript
// schemas/staffSchemas.ts
import { z } from "zod";

export const createTokenFormSchema = z.object({
  customerType: z.enum(["instant", "browser", "retail"]),
  priority: z.number().min(0).max(10).optional().default(0),
  notes: z.string().max(500).optional(),
});

export const completeServiceFormSchema = z.object({
  tokenId: z.string().uuid(),
  staffId: z.string().uuid(),
  notes: z.string().max(500).optional(),
  rating: z.number().min(1).max(5).optional(),
});

export type CreateTokenForm = z.infer<typeof createTokenFormSchema>;
export type CompleteServiceForm = z.infer<typeof completeServiceFormSchema>;
```

#### Staff Interface Hooks

```typescript
// hooks/useStaffData.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import {
  createTokenFormSchema,
  completeServiceFormSchema,
} from "../schemas/staffSchemas";

export const useStaffQueue = () => {
  return useQuery({
    queryKey: ["staff", "queue"],
    queryFn: async () => {
      const data = await queueService.getQueueStatus();
      return queueStatusSchema.parse(data);
    },
    refetchInterval: 10000, // More frequent updates for staff
  });
};

export const useCreateTokenForm = () => {
  const createTokenMutation = useCreateToken();

  const form = useForm<CreateTokenForm>({
    resolver: zodResolver(createTokenFormSchema),
    defaultValues: {
      customerType: "instant",
      priority: 0,
    },
  });

  const onSubmit = async (data: CreateTokenForm) => {
    try {
      await createTokenMutation.mutateAsync(data);
      form.reset();
    } catch (error) {
      console.error("Failed to create token:", error);
    }
  };

  return {
    form,
    onSubmit: form.handleSubmit(onSubmit),
    isLoading: createTokenMutation.isPending,
  };
};

export const useCompleteServiceForm = (tokenId: string) => {
  const completeServiceMutation = useCompleteService();

  const form = useForm<CompleteServiceForm>({
    resolver: zodResolver(completeServiceFormSchema),
    defaultValues: {
      tokenId,
      staffId: "", // Get from auth context
    },
  });

  const onSubmit = async (data: CompleteServiceForm) => {
    try {
      await completeServiceMutation.mutateAsync(data);
      form.reset();
    } catch (error) {
      console.error("Failed to complete service:", error);
    }
  };

  return {
    form,
    onSubmit: form.handleSubmit(onSubmit),
    isLoading: completeServiceMutation.isPending,
  };
};
```

#### Sample Staff Component with Forms

```tsx
const StaffInterface: React.FC = () => {
  const { data: queueData, isLoading } = useStaffQueue();
  const {
    form: createForm,
    onSubmit: onCreateSubmit,
    isLoading: isCreating,
  } = useCreateTokenForm();
  const callNextMutation = useCallNext();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  const { currentServing, nextInQueue, stats } = queueData || {};

  return (
    <div className="min-h-screen bg-gray-50">
      <StaffHeader />

      <main className="container mx-auto p-6">
        {/* Current Service Panel */}
        <CurrentServicePanel token={currentServing} />

        {/* Quick Actions */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
          <div className="flex items-center gap-4">
            <button
              onClick={() =>
                callNextMutation.mutate({ staffId: "current-staff-id" })
              }
              disabled={callNextMutation.isPending || !nextInQueue?.length}
              className="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 disabled:opacity-50"
            >
              Call Next
            </button>
          </div>
        </div>

        {/* Create Token Form */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
          <h2 className="text-xl font-semibold mb-4">Create New Token</h2>
          <form onSubmit={onCreateSubmit} className="space-y-4">
            <div>
              <label className="block text-sm font-medium mb-2">
                Customer Type
              </label>
              <select
                {...createForm.register("customerType")}
                className="w-full border border-gray-300 rounded-md px-3 py-2"
              >
                <option value="instant">Walk-in Instant</option>
                <option value="browser">Walk-in Browser</option>
                <option value="retail">Retail Customer</option>
              </select>
              {createForm.formState.errors.customerType && (
                <p className="text-red-500 text-sm mt-1">
                  {createForm.formState.errors.customerType.message}
                </p>
              )}
            </div>

            <div>
              <label className="block text-sm font-medium mb-2">
                Priority (0-10)
              </label>
              <input
                type="number"
                min="0"
                max="10"
                {...createForm.register("priority", { valueAsNumber: true })}
                className="w-full border border-gray-300 rounded-md px-3 py-2"
              />
              {createForm.formState.errors.priority && (
                <p className="text-red-500 text-sm mt-1">
                  {createForm.formState.errors.priority.message}
                </p>
              )}
            </div>

            <div>
              <label className="block text-sm font-medium mb-2">
                Notes (Optional)
              </label>
              <textarea
                {...createForm.register("notes")}
                className="w-full border border-gray-300 rounded-md px-3 py-2"
                rows={3}
              />
              {createForm.formState.errors.notes && (
                <p className="text-red-500 text-sm mt-1">
                  {createForm.formState.errors.notes.message}
                </p>
              )}
            </div>

            <button
              type="submit"
              disabled={isCreating}
              className="px-6 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 disabled:opacity-50"
            >
              {isCreating ? "Creating..." : "Create Token"}
            </button>
          </form>
        </div>

        {/* Queue Overview */}
        <QueueOverviewPanel tokens={nextInQueue || []} stats={stats} />
      </main>
    </div>
  );
};
```

### 3. Super Admin Interface

**Purpose**: System management and analytics
**Target Device**: Desktop/laptop

#### Layout Structure

```
┌─────────────────────────────────────────┐
│  [≡] Dashboard         [Notifications]  │
├─────┬───────────────────────────────────┤
│     │  Dashboard                        │
│ [≡] │  ┌─ Today's Stats ──────────────┐ │
│ │ Dashboard │ Total: 45  Served: 38 Complete │ │
│ │ Queue Mgmt│ Avg Wait: 8m  Peak: 2:30pm│ │
│ │ Staff     │ ─────────────────────────────┘ │
│ │ Analytics │  ┌─ Active Queues ─────────┐ │
│ │ Settings  │  │ Instant: 3  Browser: 1  │ │
│ │ Users     │  │ Retail: 2   Staff: 2    │ │
│ └───────────│  └─────────────────────────┘ │
│             │                             │
└─────────────┴─────────────────────────────┘
```

#### Key Features

- **Dashboard**: Real-time statistics and overview
- **Queue Management**: Monitor all queue types
- **Staff Management**: User accounts and permissions
- **Analytics**: Historical data and reports
- **System Settings**: Configuration and customization

#### Sample Component

```tsx
const SuperAdminInterface: React.FC = () => {
  return (
    <div className="min-h-screen bg-white">
      <AdminHeader />

      <div className="flex">
        <AdminSidebar />

        <main className="flex-1 p-8">
          <Routes>
            <Route path="/" element={<Dashboard />} />
            <Route path="/queue" element={<QueueManagement />} />
            <Route path="/staff" element={<StaffManagement />} />
            <Route path="/analytics" element={<Analytics />} />
            <Route path="/settings" element={<Settings />} />
          </Routes>
        </main>
      </div>
    </div>
  );
};
```

## 🔄 Real-time Updates with TanStack Query Integration

### WebSocket Implementation with Query Invalidation

```typescript
// hooks/useSocket.ts
import { useQueryClient } from "@tanstack/react-query";
import { queueStatusSchema, tokenSchema } from "../schemas/tokenSchemas";

export const useSocket = () => {
  const [socket, setSocket] = useState<Socket | null>(null);

  useEffect(() => {
    const newSocket = io(process.env.REACT_APP_SOCKET_URL);
    setSocket(newSocket);

    return () => newSocket.close();
  }, []);

  return socket;
};

// hooks/useRealTimeUpdates.ts
export const useRealTimeUpdates = () => {
  const socket = useSocket();
  const queryClient = useQueryClient();

  useEffect(() => {
    if (!socket) return;

    // Queue status updates
    socket.on("queue:updated", (data) => {
      try {
        const validatedData = queueStatusSchema.parse(data);
        queryClient.setQueryData(["queue", "status"], validatedData);
      } catch (error) {
        console.error("Invalid queue data received:", error);
        // Refetch to get valid data
        queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
      }
    });

    // Individual token updates
    socket.on("token:created", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        // Update queue status optimistically
        queryClient.setQueryData(["queue", "status"], (old: any) => {
          if (!old) return old;

          return {
            ...old,
            nextInQueue: [...old.nextInQueue, validatedToken].sort((a, b) => {
              if (a.priority !== b.priority) return b.priority - a.priority;
              return (
                new Date(a.createdAt).getTime() -
                new Date(b.createdAt).getTime()
              );
            }),
            stats: {
              ...old.stats,
              totalWaiting: old.stats.totalWaiting + 1,
            },
          };
        });
      } catch (error) {
        console.error("Invalid token data received:", error);
        queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
      }
    });

    socket.on("token:called", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        queryClient.setQueryData(["queue", "status"], (old: any) => {
          if (!old) return old;

          return {
            ...old,
            currentServing: validatedToken,
            nextInQueue: old.nextInQueue.filter(
              (t: any) => t.id !== validatedToken.id
            ),
          };
        });
      } catch (error) {
        console.error("Invalid token data received:", error);
        queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
      }
    });

    socket.on("token:completed", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        queryClient.setQueryData(["queue", "status"], (old: any) => {
          if (!old) return old;

          return {
            ...old,
            currentServing:
              old.currentServing?.id === validatedToken.id
                ? null
                : old.currentServing,
            recentlyServed: [validatedToken, ...old.recentlyServed].slice(0, 5),
            stats: {
              ...old.stats,
              totalServedToday: old.stats.totalServedToday + 1,
            },
          };
        });
      } catch (error) {
        console.error("Invalid token data received:", error);
        queryClient.invalidateQueries({ queryKey: ["queue", "status"] });
      }
    });

    // Connection status updates
    socket.on("connect", () => {
      console.log("Connected to server");
      // Refetch data when reconnected to ensure sync
      queryClient.invalidateQueries({ queryKey: ["queue"] });
    });

    socket.on("disconnect", () => {
      console.log("Disconnected from server");
    });

    return () => {
      socket.off("queue:updated");
      socket.off("token:created");
      socket.off("token:called");
      socket.off("token:completed");
      socket.off("connect");
      socket.off("disconnect");
    };
  }, [socket, queryClient]);

  return {
    isConnected: socket?.connected || false,
  };
};
```

### App Setup with QueryClient and Socket Integration

```typescript
// App.tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { queryClient } from "./lib/queryClient";

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Router>
        <Routes>
          <Route path="/display" element={<DisplayInterface />} />
          <Route path="/staff" element={<StaffInterface />} />
          <Route path="/admin" element={<AdminInterface />} />
        </Routes>
      </Router>
      {process.env.NODE_ENV === "development" && (
        <ReactQueryDevtools initialIsOpen={false} />
      )}
    </QueryClientProvider>
  );
}
```

## 📱 Responsive Design

### Breakpoints

```css
/* Mobile First Approach */
@media (min-width: 640px) {
  /* sm */
}
@media (min-width: 768px) {
  /* md */
}
@media (min-width: 1024px) {
  /* lg */
}
@media (min-width: 1280px) {
  /* xl */
}
@media (min-width: 1536px) {
  /* 2xl */
}
```

### Display Interface Adaptations

- **Large Screens (1920px+)**: Full layout with all sections
- **Medium Screens (1024px+)**: Simplified grid
- **Tablets (768px+)**: Single column layout
- **Mobile**: Not recommended for display interface

## 🎭 Animation Guidelines

### Token Transitions

```tsx
const TokenAnimation: React.FC<{ token: Token }> = ({ token }) => {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.3 }}
      className="token-card"
    >
      {token.number}
    </motion.div>
  );
};
```

### Page Transitions

```tsx
const PageTransition: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
      transition={{ duration: 0.2 }}
    >
      {children}
    </motion.div>
  );
};
```

## 🧪 Testing Strategy

### Component Testing with TanStack Query

```typescript
// __tests__/DisplayInterface.test.tsx
import { render, screen, waitFor } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { DisplayInterface } from "../DisplayInterface";
import { queueStatusSchema } from "../schemas/tokenSchemas";

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

const mockQueueData = {
  currentServing: {
    id: "1",
    number: "A001",
    customerType: "instant" as const,
    status: "called" as const,
    priority: 0,
    createdAt: "2024-01-01T10:00:00Z",
    calledAt: "2024-01-01T10:05:00Z",
  },
  nextInQueue: [],
  recentlyServed: [],
  stats: {
    totalWaiting: 0,
    averageWaitTime: 8,
    totalServedToday: 15,
  },
};

// Mock the queue service
jest.mock("../services/queueService", () => ({
  queueService: {
    getQueueStatus: jest.fn(() => Promise.resolve(mockQueueData)),
  },
}));

const renderWithQuery = (component: React.ReactElement) => {
  const queryClient = createTestQueryClient();
  return render(
    <QueryClientProvider client={queryClient}>{component}</QueryClientProvider>
  );
};

describe("DisplayInterface", () => {
  beforeEach(() => {
    // Validate mock data matches schema
    expect(() => queueStatusSchema.parse(mockQueueData)).not.toThrow();
  });

  it("displays current serving token", async () => {
    renderWithQuery(<DisplayInterface />);

    await waitFor(() => {
      expect(screen.getByText("NOW SERVING")).toBeInTheDocument();
      expect(screen.getByText("A001")).toBeInTheDocument();
    });
  });

  it("shows loading state initially", () => {
    renderWithQuery(<DisplayInterface />);
    expect(screen.getByText("Loading...")).toBeInTheDocument();
  });

  it("displays queue statistics", async () => {
    renderWithQuery(<DisplayInterface />);

    await waitFor(() => {
      expect(screen.getByText(/Total waiting: 0/)).toBeInTheDocument();
      expect(screen.getByText(/Avg wait: 8min/)).toBeInTheDocument();
    });
  });
});

// Test Zod validation
describe("tokenSchema validation", () => {
  it("validates correct token data", () => {
    const validToken = {
      id: "550e8400-e29b-41d4-a716-446655440000",
      number: "A001",
      customerType: "instant",
      status: "waiting",
      priority: 5,
      createdAt: "2024-01-01T10:00:00Z",
    };

    expect(() => tokenSchema.parse(validToken)).not.toThrow();
  });

  it("rejects invalid token data", () => {
    const invalidToken = {
      id: "invalid-uuid",
      number: "",
      customerType: "invalid",
      status: "invalid",
      priority: -1,
    };

    expect(() => tokenSchema.parse(invalidToken)).toThrow();
  });
});
```

## 🚀 Deployment Considerations

### Build Optimization

```json
{
  "scripts": {
    "build": "react-scripts build",
    "build:display": "REACT_APP_MODE=display npm run build",
    "build:staff": "REACT_APP_MODE=staff npm run build",
    "build:admin": "REACT_APP_MODE=admin npm run build"
  }
}
```

### Environment Configuration

```typescript
// config/environment.ts
export const config = {
  apiUrl: process.env.REACT_APP_API_URL || "http://localhost:3001",
  socketUrl: process.env.REACT_APP_SOCKET_URL || "http://localhost:3001",
  mode: process.env.REACT_APP_MODE || "display",
};
```

## 🔗 Integration Points

- **API Integration**: See [API Integration Guide](./api-integration-guide.md)
- **Backend API**: See [API Implementation Guide](./api-implementation-guide.md)
- **Real-time Events**: WebSocket connections for live updates
- **Authentication**: Role-based access for staff and admin interfaces

---

_This guide provides the foundation for building a modern, responsive, and user-friendly queue management system UI._
