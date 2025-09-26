# API Integration Guide

A comprehensive guide for connecting the frontend interfaces with the backend API, including real-time communication, state management, and error handling.

## 🎯 Overview

This guide demonstrates how to integrate the three frontend interfaces (Display, Staff, and Super Admin) with the backend API to create a seamless queue management experience.

## 🔗 Integration Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Display UI    │    │    Staff UI     │    │  Super Admin    │
│                 │    │                 │    │      UI         │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│   React App     │    │   React App     │    │   React App     │
│                 │    │                 │    │                 │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│  State Manager  │    │  State Manager  │    │  State Manager  │
│   (Zustand)     │    │   (Zustand)     │    │   (Zustand)     │
│                 │    │                 │    │                 │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ Socket.io Client│    │ Socket.io Client│    │ Socket.io Client│
│                 │    │                 │    │                 │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│  HTTP Client    │    │  HTTP Client    │    │  HTTP Client    │
│    (Axios)      │    │    (Axios)      │    │    (Axios)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Backend API   │
                    │                 │
                    │  Express.js +   │
                    │   Socket.io     │
                    │                 │
                    └─────────────────┘
```

## 🛠️ Setup and Configuration

### 1. HTTP Client Configuration

```typescript
// src/lib/apiClient.ts
import axios, { AxiosInstance, AxiosRequestConfig } from "axios";

class ApiClient {
  private client: AxiosInstance;
  private token: string | null = null;

  constructor() {
    this.client = axios.create({
      baseURL: process.env.REACT_APP_API_URL || "http://localhost:3001/api",
      timeout: 10000,
      headers: {
        "Content-Type": "application/json",
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor to add auth token
    this.client.interceptors.request.use(
      (config) => {
        if (this.token) {
          config.headers.Authorization = `Bearer ${this.token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for error handling
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          this.token = null;
          localStorage.removeItem("auth_token");
          window.location.href = "/login";
        }
        return Promise.reject(error);
      }
    );
  }

  setToken(token: string) {
    this.token = token;
    localStorage.setItem("auth_token", token);
  }

  removeToken() {
    this.token = null;
    localStorage.removeItem("auth_token");
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.get(url, config);
    return response.data;
  }

  async post<T>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await this.client.post(url, data, config);
    return response.data;
  }

  async patch<T>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await this.client.patch(url, data, config);
    return response.data;
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.delete(url, config);
    return response.data;
  }
}

export const apiClient = new ApiClient();
```

### 2. WebSocket Client Setup

```typescript
// src/lib/socketClient.ts
import { io, Socket } from "socket.io-client";

class SocketClient {
  private socket: Socket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;

  connect(token?: string) {
    this.socket = io(
      process.env.REACT_APP_SOCKET_URL || "http://localhost:3001",
      {
        auth: token ? { token } : undefined,
        transports: ["websocket"],
        upgrade: false,
      }
    );

    this.setupEventHandlers();
    return this.socket;
  }

  private setupEventHandlers() {
    if (!this.socket) return;

    this.socket.on("connect", () => {
      console.log("Connected to server");
      this.reconnectAttempts = 0;
    });

    this.socket.on("disconnect", (reason) => {
      console.log("Disconnected from server:", reason);

      if (reason === "io server disconnect") {
        // Server disconnected, try to reconnect
        this.handleReconnection();
      }
    });

    this.socket.on("connect_error", (error) => {
      console.error("Connection error:", error);
      this.handleReconnection();
    });
  }

  private handleReconnection() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      setTimeout(() => {
        this.socket?.connect();
      }, Math.pow(2, this.reconnectAttempts) * 1000);
    }
  }

  joinRoom(room: string) {
    this.socket?.emit("join_room", room);
  }

  leaveRoom(room: string) {
    this.socket?.emit("leave_room", room);
  }

  on<T>(event: string, callback: (data: T) => void) {
    this.socket?.on(event, callback);
  }

  off(event: string, callback?: (...args: any[]) => void) {
    this.socket?.off(event, callback);
  }

  disconnect() {
    this.socket?.disconnect();
    this.socket = null;
  }

  get connected() {
    return this.socket?.connected || false;
  }
}

export const socketClient = new SocketClient();
```

## 🗂️ Data Layer Architecture

### Data Schemas with Zod

```typescript
// src/schemas/tokenSchemas.ts
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

export const createTokenRequestSchema = z.object({
  customerType: z.enum(["instant", "browser", "retail"]),
  priority: z.number().min(0).max(10).optional().default(0),
  notes: z.string().max(500).optional(),
});

export const callNextRequestSchema = z.object({
  customerType: z.enum(["instant", "browser", "retail"]).optional(),
  staffId: z.string().uuid(),
});

export const completeServiceRequestSchema = z.object({
  tokenId: z.string().uuid(),
  staffId: z.string().uuid(),
  notes: z.string().max(500).optional(),
  rating: z.number().min(1).max(5).optional(),
});

export type Token = z.infer<typeof tokenSchema>;
export type QueueStatus = z.infer<typeof queueStatusSchema>;
export type CreateTokenRequest = z.infer<typeof createTokenRequestSchema>;
export type CallNextRequest = z.infer<typeof callNextRequestSchema>;
export type CompleteServiceRequest = z.infer<
  typeof completeServiceRequestSchema
>;
```

### TanStack Query Configuration

```typescript
// src/lib/queryClient.ts
import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 10, // 10 minutes
      refetchOnWindowFocus: false,
      retry: (failureCount, error: any) => {
        if (error?.response?.status === 404) return false;
        if (error?.response?.status === 401) return false;
        return failureCount < 3;
      },
    },
    mutations: {
      retry: (failureCount, error: any) => {
        if (error?.response?.status >= 400 && error?.response?.status < 500) {
          return false; // Don't retry client errors
        }
        return failureCount < 2;
      },
    },
  },
});

// Query Keys Factory
export const queryKeys = {
  all: ["queue"] as const,
  status: () => [...queryKeys.all, "status"] as const,
  tokens: () => [...queryKeys.all, "tokens"] as const,
  tokensByType: (type: string) => [...queryKeys.tokens(), type] as const,
  analytics: () => [...queryKeys.all, "analytics"] as const,
  staff: () => ["staff"] as const,
  staffSessions: (staffId: string) =>
    [...queryKeys.staff(), "sessions", staffId] as const,
};
```

### Minimal State Management with Zustand

```typescript
// src/store/appStore.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AppState {
  // UI State
  sidebarOpen: boolean;
  setSidebarOpen: (open: boolean) => void;

  // Connection State
  isConnected: boolean;
  setConnectionStatus: (connected: boolean) => void;

  // Notifications
  notifications: Array<{
    id: string;
    type: "success" | "error" | "warning" | "info";
    message: string;
    timestamp: Date;
  }>;
  addNotification: (
    notification: Omit<AppState["notifications"][0], "id" | "timestamp">
  ) => void;
  removeNotification: (id: string) => void;

  // Settings
  settings: {
    theme: "light" | "dark";
    autoRefresh: boolean;
    soundEnabled: boolean;
  };
  updateSettings: (settings: Partial<AppState["settings"]>) => void;
}

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({
      // UI State
      sidebarOpen: false,
      setSidebarOpen: (open) => set({ sidebarOpen: open }),

      // Connection State
      isConnected: false,
      setConnectionStatus: (connected) => set({ isConnected: connected }),

      // Notifications
      notifications: [],
      addNotification: (notification) =>
        set((state) => ({
          notifications: [
            ...state.notifications,
            {
              ...notification,
              id: crypto.randomUUID(),
              timestamp: new Date(),
            },
          ],
        })),
      removeNotification: (id) =>
        set((state) => ({
          notifications: state.notifications.filter((n) => n.id !== id),
        })),

      // Settings
      settings: {
        theme: "light",
        autoRefresh: true,
        soundEnabled: true,
      },
      updateSettings: (newSettings) =>
        set((state) => ({
          settings: { ...state.settings, ...newSettings },
        })),
    }),
    {
      name: "app-storage",
      partialize: (state) => ({
        settings: state.settings,
        sidebarOpen: state.sidebarOpen,
      }),
    }
  )
);
```

### Authentication Store

```typescript
// src/store/authStore.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface User {
  id: string;
  username: string;
  email: string;
  role: "staff" | "admin" | "super_admin";
  isActive: boolean;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;

  login: (token: string, user: User) => void;
  logout: () => void;
  updateUser: (updates: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: (token, user) =>
        set({
          token,
          user,
          isAuthenticated: true,
        }),

      logout: () =>
        set({
          token: null,
          user: null,
          isAuthenticated: false,
        }),

      updateUser: (updates) =>
        set((state) => ({
          user: state.user ? { ...state.user, ...updates } : null,
        })),
    }),
    {
      name: "auth-storage",
      partialize: (state) => ({
        token: state.token,
        user: state.user,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);
```

## 🔧 API Service Layer with Validation

### Queue Service with Zod Validation

```typescript
// src/services/queueService.ts
import { apiClient } from "../lib/apiClient";
import {
  queueStatusSchema,
  tokenSchema,
  createTokenRequestSchema,
  callNextRequestSchema,
  completeServiceRequestSchema,
  type QueueStatus,
  type Token,
  type CreateTokenRequest,
  type CallNextRequest,
  type CompleteServiceRequest,
} from "../schemas/tokenSchemas";

class QueueService {
  async getQueueStatus(): Promise<QueueStatus> {
    try {
      const response = await apiClient.get("/queue/status");
      return queueStatusSchema.parse(response);
    } catch (error) {
      console.error("Failed to fetch queue status:", error);
      throw error;
    }
  }

  async createToken(request: CreateTokenRequest): Promise<Token> {
    try {
      // Validate request data
      const validatedRequest = createTokenRequestSchema.parse(request);

      const response = await apiClient.post("/tokens", validatedRequest);
      return tokenSchema.parse(response);
    } catch (error) {
      console.error("Failed to create token:", error);
      throw error;
    }
  }

  async callNext(request: CallNextRequest): Promise<Token> {
    try {
      // Validate request data
      const validatedRequest = callNextRequestSchema.parse(request);

      const response = await apiClient.post(
        "/queue/call-next",
        validatedRequest
      );
      return tokenSchema.parse(response);
    } catch (error) {
      console.error("Failed to call next token:", error);
      throw error;
    }
  }

  async completeService(request: CompleteServiceRequest): Promise<Token> {
    try {
      // Validate request data
      const validatedRequest = completeServiceRequestSchema.parse(request);

      const response = await apiClient.post(
        "/queue/complete-service",
        validatedRequest
      );
      return tokenSchema.parse(response);
    } catch (error) {
      console.error("Failed to complete service:", error);
      throw error;
    }
  }

  async updateToken(tokenId: string, updates: Partial<Token>): Promise<Token> {
    try {
      const response = await apiClient.patch(`/tokens/${tokenId}`, updates);
      return tokenSchema.parse(response);
    } catch (error) {
      console.error("Failed to update token:", error);
      throw error;
    }
  }

  async getTokens(query?: {
    status?: string;
    customerType?: string;
    limit?: number;
    offset?: number;
  }): Promise<Token[]> {
    try {
      const response = await apiClient.get("/tokens", { params: query });
      return response.map((token: any) => tokenSchema.parse(token));
    } catch (error) {
      console.error("Failed to fetch tokens:", error);
      throw error;
    }
  }

  async deleteToken(tokenId: string): Promise<void> {
    try {
      await apiClient.delete(`/tokens/${tokenId}`);
    } catch (error) {
      console.error("Failed to delete token:", error);
      throw error;
    }
  }
}

export const queueService = new QueueService();
```

### TanStack Query Hooks

```typescript
// src/hooks/useQueueData.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { queueService } from "../services/queueService";
import { queryKeys } from "../lib/queryClient";
import { useAppStore } from "../store/appStore";
import type {
  CreateTokenRequest,
  CallNextRequest,
  CompleteServiceRequest,
} from "../schemas/tokenSchemas";

export const useQueueStatus = () => {
  return useQuery({
    queryKey: queryKeys.status(),
    queryFn: queueService.getQueueStatus,
    refetchInterval: 30000, // Fallback refresh every 30s
    staleTime: 1000 * 60 * 2, // 2 minutes
  });
};

export const useTokens = (query?: {
  status?: string;
  customerType?: string;
}) => {
  return useQuery({
    queryKey: [...queryKeys.tokens(), query],
    queryFn: () => queueService.getTokens(query),
    enabled: !!query,
  });
};

export const useCreateToken = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (request: CreateTokenRequest) =>
      queueService.createToken(request),
    onSuccess: (newToken) => {
      // Optimistically update the queue status
      queryClient.setQueryData(queryKeys.status(), (old: any) => {
        if (!old) return old;
        return {
          ...old,
          nextInQueue: [...old.nextInQueue, newToken].sort((a, b) => {
            if (a.priority !== b.priority) return b.priority - a.priority;
            return (
              new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
            );
          }),
          stats: {
            ...old.stats,
            totalWaiting: old.stats.totalWaiting + 1,
          },
        };
      });

      // Invalidate related queries
      queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });

      addNotification({
        type: "success",
        message: `Token ${newToken.number} created successfully`,
      });
    },
    onError: (error: any) => {
      addNotification({
        type: "error",
        message: error.response?.data?.message || "Failed to create token",
      });
    },
  });
};

export const useCallNext = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (request: CallNextRequest) => queueService.callNext(request),
    onSuccess: (calledToken) => {
      // Optimistically update the queue status
      queryClient.setQueryData(queryKeys.status(), (old: any) => {
        if (!old) return old;
        return {
          ...old,
          currentServing: calledToken,
          nextInQueue: old.nextInQueue.filter(
            (t: any) => t.id !== calledToken.id
          ),
        };
      });

      queryClient.invalidateQueries({ queryKey: queryKeys.status() });

      addNotification({
        type: "success",
        message: `Token ${calledToken.number} called to service`,
      });
    },
    onError: (error: any) => {
      addNotification({
        type: "error",
        message: error.response?.data?.message || "Failed to call next token",
      });
    },
  });
};

export const useCompleteService = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (request: CompleteServiceRequest) =>
      queueService.completeService(request),
    onSuccess: (completedToken) => {
      // Optimistically update the queue status
      queryClient.setQueryData(queryKeys.status(), (old: any) => {
        if (!old) return old;
        return {
          ...old,
          currentServing:
            old.currentServing?.id === completedToken.id
              ? null
              : old.currentServing,
          recentlyServed: [completedToken, ...old.recentlyServed].slice(0, 5),
          stats: {
            ...old.stats,
            totalServedToday: old.stats.totalServedToday + 1,
          },
        };
      });

      queryClient.invalidateQueries({ queryKey: queryKeys.status() });

      addNotification({
        type: "success",
        message: `Service completed for token ${completedToken.number}`,
      });
    },
    onError: (error: any) => {
      addNotification({
        type: "error",
        message: error.response?.data?.message || "Failed to complete service",
      });
    },
  });
};

export const useUpdateToken = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: ({ tokenId, updates }: { tokenId: string; updates: any }) =>
      queueService.updateToken(tokenId, updates),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });

      addNotification({
        type: "success",
        message: "Token updated successfully",
      });
    },
    onError: (error: any) => {
      addNotification({
        type: "error",
        message: error.response?.data?.message || "Failed to update token",
      });
    },
  });
};

export const useDeleteToken = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (tokenId: string) => queueService.deleteToken(tokenId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });

      addNotification({
        type: "success",
        message: "Token deleted successfully",
      });
    },
    onError: (error: any) => {
      addNotification({
        type: "error",
        message: error.response?.data?.message || "Failed to delete token",
      });
    },
  });
};
```

### Authentication Service

```typescript
// src/services/authService.ts
import { apiClient } from "../lib/apiClient";
import { useAuthStore } from "../store/authStore";

export interface LoginRequest {
  username: string;
  password: string;
}

class AuthService {
  async login(credentials: LoginRequest) {
    try {
      const response = await apiClient.post("/auth/login", credentials);
      const { token, user } = response;

      apiClient.setToken(token);
      useAuthStore.getState().login(token, user);

      return response;
    } catch (error) {
      console.error("Login failed:", error);
      throw error;
    }
  }

  async logout() {
    try {
      await apiClient.post("/auth/logout");
    } catch (error) {
      console.error("Logout request failed:", error);
    } finally {
      apiClient.removeToken();
      useAuthStore.getState().logout();
    }
  }

  async getCurrentUser() {
    try {
      const user = await apiClient.get("/auth/me");
      useAuthStore.getState().updateUser(user);
      return user;
    } catch (error) {
      console.error("Failed to fetch current user:", error);
      throw error;
    }
  }

  initializeAuth() {
    const { token } = useAuthStore.getState();
    if (token) {
      apiClient.setToken(token);
      this.getCurrentUser().catch(() => {
        // Token might be expired, logout
        this.logout();
      });
    }
  }
}

export const authService = new AuthService();
```

## 🔄 Real-time Integration with TanStack Query

### Enhanced WebSocket Integration

```typescript
// src/hooks/useRealTimeUpdates.ts
import { useEffect } from "react";
import { useQueryClient } from "@tanstack/react-query";
import { socketClient } from "../lib/socketClient";
import { useAuthStore } from "../store/authStore";
import { useAppStore } from "../store/appStore";
import { queryKeys } from "../lib/queryClient";
import { queueStatusSchema, tokenSchema } from "../schemas/tokenSchemas";

export const useRealTimeUpdates = () => {
  const { token, user } = useAuthStore();
  const { setConnectionStatus, addNotification } = useAppStore();
  const queryClient = useQueryClient();

  useEffect(() => {
    if (!token) return;

    // Connect to socket
    const socket = socketClient.connect(token);

    // Connection status handlers
    socket.on("connect", () => {
      setConnectionStatus(true);
      addNotification({
        type: "success",
        message: "Connected to real-time updates",
      });

      // Refetch data when reconnected to ensure sync
      queryClient.invalidateQueries({ queryKey: queryKeys.all });
    });

    socket.on("disconnect", () => {
      setConnectionStatus(false);
      addNotification({
        type: "warning",
        message: "Disconnected from real-time updates",
      });
    });

    socket.on("connect_error", (error) => {
      console.error("Socket connection error:", error);
      addNotification({
        type: "error",
        message: "Failed to connect to real-time updates",
      });
    });

    // Queue status updates
    socket.on("queue:updated", (data) => {
      try {
        const validatedData = queueStatusSchema.parse(data);
        queryClient.setQueryData(queryKeys.status(), validatedData);
      } catch (error) {
        console.error("Invalid queue data received:", error);
        // Refetch to get valid data
        queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      }
    });

    // Individual token updates
    socket.on("token:created", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        // Update queue status optimistically
        queryClient.setQueryData(queryKeys.status(), (old: any) => {
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

        // Invalidate tokens list
        queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });

        addNotification({
          type: "info",
          message: `New token ${validatedToken.number} created`,
        });
      } catch (error) {
        console.error("Invalid token data received:", error);
        queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      }
    });

    socket.on("token:called", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        queryClient.setQueryData(queryKeys.status(), (old: any) => {
          if (!old) return old;

          return {
            ...old,
            currentServing: validatedToken,
            nextInQueue: old.nextInQueue.filter(
              (t: any) => t.id !== validatedToken.id
            ),
          };
        });

        queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });

        // Play notification sound for staff
        if (user?.role === "staff" && validatedToken.servedBy === user.id) {
          addNotification({
            type: "success",
            message: `Token ${validatedToken.number} is ready for service`,
          });
        }
      } catch (error) {
        console.error("Invalid token data received:", error);
        queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      }
    });

    socket.on("token:completed", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        queryClient.setQueryData(queryKeys.status(), (old: any) => {
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

        queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });
      } catch (error) {
        console.error("Invalid token data received:", error);
        queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      }
    });

    socket.on("token:updated", (tokenData) => {
      try {
        const validatedToken = tokenSchema.parse(tokenData);

        // Update specific token queries
        queryClient.invalidateQueries({ queryKey: queryKeys.tokens() });
        queryClient.invalidateQueries({ queryKey: queryKeys.status() });
      } catch (error) {
        console.error("Invalid token data received:", error);
      }
    });

    socket.on("system:maintenance", (message) => {
      addNotification({
        type: "warning",
        message: `System maintenance: ${message}`,
      });
    });

    socket.on("queue:paused", (data) => {
      addNotification({
        type: "warning",
        message: `Queue paused for ${data.customerType} customers`,
      });
    });

    socket.on("queue:resumed", (data) => {
      addNotification({
        type: "success",
        message: `Queue resumed for ${data.customerType} customers`,
      });
    });

    // Join appropriate rooms based on user role
    if (user) {
      socket.emit("join_room", `${user.role}_updates`);
      socket.emit("join_room", "queue_updates");
    }

    // Cleanup on unmount
    return () => {
      socket.off("connect");
      socket.off("disconnect");
      socket.off("connect_error");
      socket.off("queue:updated");
      socket.off("token:created");
      socket.off("token:called");
      socket.off("token:completed");
      socket.off("token:updated");
      socket.off("system:maintenance");
      socket.off("queue:paused");
      socket.off("queue:resumed");

      socketClient.disconnect();
      setConnectionStatus(false);
    };
  }, [token, user, queryClient, setConnectionStatus, addNotification]);

  return {
    isConnected: useAppStore((state) => state.isConnected),
  };
};
```

### Display Interface with TanStack Query

```typescript
// src/components/Display/DisplayInterface.tsx
import React from "react";
import { motion, AnimatePresence } from "framer-motion";
import { useQueueStatus } from "../../hooks/useQueueData";
import { useRealTimeUpdates } from "../../hooks/useRealTimeUpdates";
import { useAppStore } from "../../store/appStore";
import { LoadingSpinner } from "../common/LoadingSpinner";
import { ErrorMessage } from "../common/ErrorMessage";

export const DisplayInterface: React.FC = () => {
  const { data: queueStatus, isLoading, error, isError } = useQueueStatus();
  const { isConnected } = useRealTimeUpdates();
  const { notifications } = useAppStore();

  if (isLoading) {
    return (
      <div className="min-h-screen bg-white flex items-center justify-center">
        <LoadingSpinner size="lg" />
      </div>
    );
  }

  if (isError) {
    return (
      <div className="min-h-screen bg-white flex items-center justify-center">
        <ErrorMessage
          message="Failed to load queue data"
          error={error}
          retry={() => window.location.reload()}
        />
      </div>
    );
  }

  const { currentServing, nextInQueue, recentlyServed, stats } =
    queueStatus || {};

  return (
    <div className="min-h-screen bg-white p-8 relative">
      {/* Connection indicator */}
      <div
        className={`absolute top-4 right-4 w-4 h-4 rounded-full ${
          isConnected ? "bg-green-500" : "bg-red-500"
        }`}
        title={isConnected ? "Connected" : "Disconnected"}
      />

      {/* Notifications */}
      <NotificationPanel notifications={notifications} />

      {/* Header */}
      <header className="text-center mb-12">
        <h1 className="text-4xl font-light text-gray-800">
          Queue Management System
        </h1>
        <p className="text-gray-600 mt-2">
          Total waiting: {stats?.totalWaiting || 0} | Avg wait:{" "}
          {stats?.averageWaitTime || 0}min | Served today:{" "}
          {stats?.totalServedToday || 0}
        </p>
      </header>

      {/* Current Serving */}
      <section className="text-center mb-16">
        <h2 className="text-3xl font-light text-gray-600 mb-8">NOW SERVING</h2>
        <AnimatePresence mode="wait">
          <motion.div
            key={currentServing?.id || "empty"}
            initial={{ opacity: 0, scale: 0.8 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.8 }}
            transition={{ duration: 0.5, ease: "easeInOut" }}
            className="text-8xl font-bold text-gray-900"
          >
            {currentServing?.number || "---"}
          </motion.div>
        </AnimatePresence>
        {currentServing && (
          <motion.div
            initial={{ opacity: 0, y: 10 }}
            animate={{ opacity: 1, y: 0 }}
            className="mt-4"
          >
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
            {currentServing.priority > 0 && (
              <span className="ml-2 px-2 py-1 bg-red-100 text-red-800 rounded-full text-sm">
                Priority
              </span>
            )}
          </motion.div>
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

const QueueSection: React.FC<{
  title: string;
  tokens: Array<{
    id: string;
    number: string;
    customerType: string;
    priority?: number;
    createdAt: string;
  }>;
}> = ({ title, tokens }) => (
  <section>
    <h3 className="text-2xl font-light text-gray-700 mb-6 text-center">
      {title}
    </h3>
    <div className="space-y-4">
      <AnimatePresence>
        {tokens.slice(0, 5).map((token, index) => (
          <motion.div
            key={token.id}
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -20 }}
            transition={{ duration: 0.3, delay: index * 0.1 }}
            className="flex items-center justify-between p-4 bg-gray-50 rounded-lg"
          >
            <div className="flex items-center space-x-2">
              <span className="text-xl font-semibold text-gray-800">
                {token.number}
              </span>
              {token.priority && token.priority > 0 && (
                <span className="px-2 py-1 bg-red-100 text-red-800 rounded-full text-xs">
                  Priority
                </span>
              )}
            </div>
            <div className="flex items-center space-x-2">
              <span
                className={`px-3 py-1 rounded-full text-sm text-white ${
                  token.customerType === "instant"
                    ? "bg-blue-500"
                    : token.customerType === "browser"
                    ? "bg-green-500"
                    : "bg-purple-500"
                }`}
              >
                {token.customerType}
              </span>
              {title === "Next in Queue" && (
                <span className="text-xs text-gray-500">
                  {new Date(token.createdAt).toLocaleTimeString()}
                </span>
              )}
            </div>
          </motion.div>
        ))}
      </AnimatePresence>
      {tokens.length === 0 && (
        <div className="text-center py-8 text-gray-500">
          No tokens {title.toLowerCase()}
        </div>
      )}
    </div>
  </section>
);

const NotificationPanel: React.FC<{ notifications: any[] }> = ({
  notifications,
}) => (
  <div className="fixed top-4 left-4 space-y-2 z-50">
    <AnimatePresence>
      {notifications.slice(-3).map((notification) => (
        <motion.div
          key={notification.id}
          initial={{ opacity: 0, x: -100 }}
          animate={{ opacity: 1, x: 0 }}
          exit={{ opacity: 0, x: -100 }}
          className={`px-4 py-2 rounded-lg text-white text-sm max-w-xs ${
            notification.type === "success"
              ? "bg-green-500"
              : notification.type === "error"
              ? "bg-red-500"
              : notification.type === "warning"
              ? "bg-yellow-500"
              : "bg-blue-500"
          }`}
        >
          {notification.message}
        </motion.div>
      ))}
    </AnimatePresence>
  </div>
);
```

### Staff Interface with Forms and Mutations

```typescript
// src/components/Staff/StaffInterface.tsx
import React from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import {
  useQueueStatus,
  useCreateToken,
  useCallNext,
  useCompleteService,
} from "../../hooks/useQueueData";
import { useRealTimeUpdates } from "../../hooks/useRealTimeUpdates";
import { useAuthStore } from "../../store/authStore";
import { useAppStore } from "../../store/appStore";
import {
  createTokenRequestSchema,
  type CreateTokenRequest,
} from "../../schemas/tokenSchemas";
import { LoadingSpinner } from "../common/LoadingSpinner";
import { TokenCard } from "../common/TokenCard";

export const StaffInterface: React.FC = () => {
  const { user } = useAuthStore();
  const { data: queueData, isLoading } = useQueueStatus();
  const { isConnected } = useRealTimeUpdates();
  const { notifications } = useAppStore();

  const createTokenMutation = useCreateToken();
  const callNextMutation = useCallNext();
  const completeServiceMutation = useCompleteService();

  const createTokenForm = useForm<CreateTokenRequest>({
    resolver: zodResolver(createTokenRequestSchema),
    defaultValues: {
      customerType: "instant",
      priority: 0,
    },
  });

  const onCreateToken = async (data: CreateTokenRequest) => {
    await createTokenMutation.mutateAsync(data);
    createTokenForm.reset();
  };

  const handleCallNext = (customerType?: "instant" | "browser" | "retail") => {
    if (!user) return;
    callNextMutation.mutate({
      staffId: user.id,
      ...(customerType && { customerType }),
    });
  };

  const handleCompleteService = (tokenId: string, notes?: string) => {
    if (!user) return;
    completeServiceMutation.mutate({
      tokenId,
      staffId: user.id,
      notes,
    });
  };

  if (isLoading) {
    return (
      <div className="min-h-screen bg-gray-50 flex items-center justify-center">
        <LoadingSpinner size="lg" />
      </div>
    );
  }

  const { currentServing, nextInQueue, stats } = queueData || {};

  return (
    <div className="min-h-screen bg-gray-50">
      <StaffHeader user={user} />

      <main className="container mx-auto p-6 max-w-4xl">
        {/* Current Service Panel */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
          <h2 className="text-xl font-semibold mb-4">Current Service</h2>

          {currentServing ? (
            <div className="flex items-center justify-between">
              <div>
                <span className="text-3xl font-bold text-blue-600">
                  {currentServing.number}
                </span>
                <span className="ml-4 px-3 py-1 bg-blue-100 text-blue-800 rounded-full text-sm">
                  {currentServing.customerType}
                </span>
              </div>
              <button
                onClick={handleCompleteService}
                disabled={isProcessing}
                className="px-6 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 disabled:opacity-50"
              >
                Complete Service
              </button>
            </div>
          ) : (
            <div className="text-center py-8 text-gray-500">
              No customer currently being served
            </div>
          )}
        </div>

        {/* Actions Panel */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
          <div className="flex items-center space-x-4">
            <select
              value={selectedCustomerType}
              onChange={(e) => setSelectedCustomerType(e.target.value as any)}
              className="border border-gray-300 rounded-md px-3 py-2"
            >
              <option value="instant">Walk-in Instant</option>
              <option value="browser">Walk-in Browser</option>
              <option value="retail">Retail Customer</option>
            </select>

            <button
              onClick={handleCallNext}
              disabled={isProcessing || nextInQueue.length === 0}
              className="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 disabled:opacity-50"
            >
              Call Next
            </button>

            <button
              onClick={handleCreateToken}
              disabled={isProcessing}
              className="px-6 py-2 bg-gray-600 text-white rounded-md hover:bg-gray-700 disabled:opacity-50"
            >
              Add Token
            </button>
          </div>
        </div>

        {/* Queue Overview */}
        <div className="bg-white rounded-lg shadow-md p-6">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-xl font-semibold">Queue Overview</h2>
            <span className="text-sm text-gray-600">
              {nextInQueue.length} waiting | Avg wait: {stats.averageWaitTime}
              min
            </span>
          </div>

          <div className="space-y-2">
            {nextInQueue.slice(0, 10).map((token) => (
              <div
                key={token.id}
                className="flex items-center justify-between p-3 bg-gray-50 rounded-md"
              >
                <div className="flex items-center space-x-3">
                  <span className="font-semibold">{token.number}</span>
                  <span
                    className={`px-2 py-1 rounded-full text-xs text-white ${
                      token.customerType === "instant"
                        ? "bg-blue-500"
                        : token.customerType === "browser"
                        ? "bg-green-500"
                        : "bg-purple-500"
                    }`}
                  >
                    {token.customerType}
                  </span>
                  {token.priority > 0 && (
                    <span className="px-2 py-1 bg-red-100 text-red-800 rounded-full text-xs">
                      Priority
                    </span>
                  )}
                </div>
                <span className="text-sm text-gray-500">
                  {new Date(token.createdAt).toLocaleTimeString()}
                </span>
              </div>
            ))}
          </div>
        </div>
      </main>
    </div>
  );
};

const StaffHeader: React.FC<{ user: any }> = ({ user }) => (
  <header className="bg-white shadow-sm border-b">
    <div className="container mx-auto px-6 py-4">
      <div className="flex items-center justify-between">
        <h1 className="text-xl font-semibold text-gray-800">Staff Dashboard</h1>
        <div className="flex items-center space-x-4">
          <span className="text-sm text-gray-600">
            Welcome, {user?.username}
          </span>
          <button className="text-sm text-blue-600 hover:text-blue-800">
            Logout
          </button>
        </div>
      </div>
    </div>
  </header>
);
```

## 🚨 Error Handling

### Global Error Handler

```typescript
// src/lib/errorHandler.ts
import { toast } from "react-hot-toast";

export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = "AppError";
  }
}

export const handleError = (error: any) => {
  console.error("Error:", error);

  if (error.response) {
    // HTTP error response
    const status = error.response.status;
    const message = error.response.data?.message || error.message;

    switch (status) {
      case 400:
        toast.error(`Bad Request: ${message}`);
        break;
      case 401:
        toast.error("Unauthorized access");
        break;
      case 403:
        toast.error("Access forbidden");
        break;
      case 404:
        toast.error("Resource not found");
        break;
      case 500:
        toast.error("Server error. Please try again later.");
        break;
      default:
        toast.error(message || "An unexpected error occurred");
    }
  } else if (error.code === "NETWORK_ERROR") {
    toast.error("Network error. Please check your connection.");
  } else {
    toast.error(error.message || "An unexpected error occurred");
  }
};

// React Error Boundary
export class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("Error boundary caught an error:", error, errorInfo);
    handleError(error);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen flex items-center justify-center bg-gray-50">
          <div className="text-center">
            <h1 className="text-xl font-semibold text-gray-900 mb-2">
              Something went wrong
            </h1>
            <p className="text-gray-600 mb-4">
              Please refresh the page and try again.
            </p>
            <button
              onClick={() => window.location.reload()}
              className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700"
            >
              Refresh Page
            </button>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Retry Logic

```typescript
// src/lib/retryLogic.ts
export const retryOperation = async <T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> => {
  let lastError: Error;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;

      if (attempt === maxRetries) {
        throw lastError;
      }

      // Exponential backoff
      await new Promise((resolve) =>
        setTimeout(resolve, delay * Math.pow(2, attempt - 1))
      );
    }
  }

  throw lastError!;
};
```

## 📊 Performance Optimization

### Request Debouncing

```typescript
// src/hooks/useDebounce.ts
import { useCallback, useRef } from "react";

export const useDebounce = <T extends (...args: any[]) => any>(
  callback: T,
  delay: number
): T => {
  const timeoutRef = useRef<NodeJS.Timeout>();

  return useCallback(
    (...args: Parameters<T>) => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }

      timeoutRef.current = setTimeout(() => {
        callback(...args);
      }, delay);
    },
    [callback, delay]
  ) as T;
};
```

### Data Caching

```typescript
// src/hooks/useCache.ts
import { useRef, useCallback } from "react";

interface CacheItem<T> {
  data: T;
  timestamp: number;
  ttl: number;
}

export const useCache = <T>(defaultTTL: number = 5 * 60 * 1000) => {
  const cache = useRef<Map<string, CacheItem<T>>>(new Map());

  const get = useCallback((key: string): T | null => {
    const item = cache.current.get(key);

    if (!item) return null;

    if (Date.now() - item.timestamp > item.ttl) {
      cache.current.delete(key);
      return null;
    }

    return item.data;
  }, []);

  const set = useCallback(
    (key: string, data: T, ttl: number = defaultTTL) => {
      cache.current.set(key, {
        data,
        timestamp: Date.now(),
        ttl,
      });
    },
    [defaultTTL]
  );

  const clear = useCallback((key?: string) => {
    if (key) {
      cache.current.delete(key);
    } else {
      cache.current.clear();
    }
  }, []);

  return { get, set, clear };
};
```

## 🧪 Testing Integration with TanStack Query

### Test Setup Utilities

```typescript
// src/__tests__/utils/testUtils.tsx
import React from "react";
import { render, RenderOptions } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { queueStatusSchema, tokenSchema } from "../../schemas/tokenSchemas";

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
    logger: {
      log: console.log,
      warn: console.warn,
      error: () => {}, // Silence errors in tests
    },
  });

interface CustomRenderOptions extends Omit<RenderOptions, "wrapper"> {
  queryClient?: QueryClient;
}

export const renderWithQuery = (
  ui: React.ReactElement,
  options: CustomRenderOptions = {}
) => {
  const { queryClient = createTestQueryClient(), ...renderOptions } = options;

  const Wrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );

  return {
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
    queryClient,
  };
};

// Mock data that conforms to Zod schemas
export const mockQueueData = {
  currentServing: {
    id: "550e8400-e29b-41d4-a716-446655440000",
    number: "A001",
    customerType: "instant" as const,
    status: "called" as const,
    priority: 0,
    createdAt: "2024-01-01T10:00:00Z",
    calledAt: "2024-01-01T10:05:00Z",
  },
  nextInQueue: [
    {
      id: "550e8400-e29b-41d4-a716-446655440001",
      number: "A002",
      customerType: "browser" as const,
      status: "waiting" as const,
      priority: 0,
      createdAt: "2024-01-01T10:01:00Z",
    },
  ],
  recentlyServed: [],
  stats: {
    totalWaiting: 1,
    averageWaitTime: 5,
    totalServedToday: 15,
  },
};

// Validate mock data against schemas
beforeAll(() => {
  expect(() => queueStatusSchema.parse(mockQueueData)).not.toThrow();
  expect(() => tokenSchema.parse(mockQueueData.currentServing)).not.toThrow();
});
```

### API Mocking with MSW

```typescript
// src/__tests__/mocks/handlers.ts
import { rest } from "msw";
import { mockQueueData } from "../utils/testUtils";

export const handlers = [
  rest.get("/api/queue/status", (req, res, ctx) => {
    return res(ctx.json(mockQueueData));
  }),

  rest.post("/api/tokens", (req, res, ctx) => {
    const newToken = {
      id: "550e8400-e29b-41d4-a716-446655440002",
      number: "A003",
      customerType: "instant",
      status: "waiting",
      priority: 0,
      createdAt: new Date().toISOString(),
    };
    return res(ctx.json(newToken));
  }),

  rest.post("/api/queue/call-next", (req, res, ctx) => {
    return res(
      ctx.json({
        ...mockQueueData.nextInQueue[0],
        status: "called",
        calledAt: new Date().toISOString(),
      })
    );
  }),

  rest.post("/api/queue/complete-service", (req, res, ctx) => {
    return res(
      ctx.json({
        ...mockQueueData.currentServing,
        status: "completed",
        completedAt: new Date().toISOString(),
      })
    );
  }),
];

// Error handlers for testing error states
export const errorHandlers = [
  rest.get("/api/queue/status", (req, res, ctx) => {
    return res(ctx.status(500), ctx.json({ message: "Internal server error" }));
  }),
];
```

### Component Testing with TanStack Query

```typescript
// src/__tests__/components/DisplayInterface.test.tsx
import { screen, waitFor } from "@testing-library/react";
import { rest } from "msw";
import { setupServer } from "msw/node";
import { DisplayInterface } from "../../components/Display/DisplayInterface";
import { renderWithQuery, mockQueueData } from "../utils/testUtils";
import { handlers, errorHandlers } from "../mocks/handlers";

const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe("DisplayInterface", () => {
  it("displays loading state initially", () => {
    renderWithQuery(<DisplayInterface />);
    expect(screen.getByText("Loading...")).toBeInTheDocument();
  });

  it("displays queue data after loading", async () => {
    renderWithQuery(<DisplayInterface />);

    await waitFor(() => {
      expect(screen.getByText("NOW SERVING")).toBeInTheDocument();
      expect(screen.getByText("A001")).toBeInTheDocument();
    });

    expect(screen.getByText(/Total waiting: 1/)).toBeInTheDocument();
    expect(screen.getByText(/Avg wait: 5min/)).toBeInTheDocument();
  });

  it("displays error state when API fails", async () => {
    server.use(...errorHandlers);

    renderWithQuery(<DisplayInterface />);

    await waitFor(() => {
      expect(screen.getByText("Failed to load queue data")).toBeInTheDocument();
    });
  });

  it("updates data when real-time events occur", async () => {
    const { queryClient } = renderWithQuery(<DisplayInterface />);

    // Wait for initial load
    await waitFor(() => {
      expect(screen.getByText("A001")).toBeInTheDocument();
    });

    // Simulate real-time update
    const newQueueData = {
      ...mockQueueData,
      currentServing: {
        ...mockQueueData.currentServing,
        number: "A002",
      },
    };

    queryClient.setQueryData(["queue", "status"], newQueueData);

    await waitFor(() => {
      expect(screen.getByText("A002")).toBeInTheDocument();
    });
  });
});
```

### Form Testing with React Hook Form

```typescript
// src/__tests__/components/StaffInterface.test.tsx
import { screen, fireEvent, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { StaffInterface } from "../../components/Staff/StaffInterface";
import { renderWithQuery } from "../utils/testUtils";
import { useAuthStore } from "../../store/authStore";

// Mock auth store
jest.mock("../../store/authStore");
const mockUseAuthStore = useAuthStore as jest.MockedFunction<
  typeof useAuthStore
>;

describe("StaffInterface", () => {
  beforeEach(() => {
    mockUseAuthStore.mockReturnValue({
      user: {
        id: "staff-1",
        username: "teststaff",
        role: "staff",
      },
      token: "mock-token",
    } as any);
  });

  it("allows creating a new token with form validation", async () => {
    const user = userEvent.setup();
    renderWithQuery(<StaffInterface />);

    await waitFor(() => {
      expect(screen.getByText("Create New Token")).toBeInTheDocument();
    });

    // Fill out form
    const customerTypeSelect = screen.getByLabelText("Customer Type");
    const priorityInput = screen.getByLabelText("Priority (0-10)");
    const notesTextarea = screen.getByLabelText("Notes (Optional)");

    await user.selectOptions(customerTypeSelect, "retail");
    await user.clear(priorityInput);
    await user.type(priorityInput, "5");
    await user.type(notesTextarea, "VIP customer");

    // Submit form
    const submitButton = screen.getByRole("button", { name: /create token/i });
    await user.click(submitButton);

    await waitFor(() => {
      expect(
        screen.getByText(/token.*created successfully/i)
      ).toBeInTheDocument();
    });
  });

  it("shows validation errors for invalid form data", async () => {
    const user = userEvent.setup();
    renderWithQuery(<StaffInterface />);

    await waitFor(() => {
      expect(screen.getByText("Create New Token")).toBeInTheDocument();
    });

    const priorityInput = screen.getByLabelText("Priority (0-10)");
    await user.clear(priorityInput);
    await user.type(priorityInput, "15"); // Invalid priority

    const submitButton = screen.getByRole("button", { name: /create token/i });
    await user.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/must be at most 10/i)).toBeInTheDocument();
    });
  });

  it("handles call next functionality", async () => {
    const user = userEvent.setup();
    renderWithQuery(<StaffInterface />);

    await waitFor(() => {
      expect(screen.getByText("Call Next (Any Type)")).toBeInTheDocument();
    });

    const callNextButton = screen.getByRole("button", { name: /call next/i });
    await user.click(callNextButton);

    await waitFor(() => {
      expect(screen.getByText(/token.*called to service/i)).toBeInTheDocument();
    });
  });
});
```

### Hook Testing

```typescript
// src/__tests__/hooks/useQueueData.test.tsx
import { renderHook, waitFor } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useQueueStatus, useCreateToken } from "../../hooks/useQueueData";
import { mockQueueData } from "../utils/testUtils";

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false }, mutations: { retry: false } },
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

describe("useQueueStatus", () => {
  it("fetches and returns queue data", async () => {
    const { result } = renderHook(() => useQueueStatus(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual(mockQueueData);
  });
});

describe("useCreateToken", () => {
  it("creates a token and updates cache", async () => {
    const { result } = renderHook(() => useCreateToken(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.mutate).toBeDefined();
    });

    // Test mutation
    result.current.mutate({
      customerType: "instant",
      priority: 0,
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });
  });
});
```

## 🚀 Deployment Configuration

### Environment Variables

```bash
# Frontend (.env)
REACT_APP_API_URL=https://api.yourdomain.com/api
REACT_APP_SOCKET_URL=https://api.yourdomain.com
REACT_APP_MODE=production
```

### Build Configuration

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "build:display": "REACT_APP_BUILD_TARGET=display npm run build",
    "build:staff": "REACT_APP_BUILD_TARGET=staff npm run build",
    "build:admin": "REACT_APP_BUILD_TARGET=admin npm run build"
  }
}
```

---

_This integration guide provides a complete blueprint for connecting your queue management system's frontend and backend components with proper error handling, real-time updates, and performance optimization._
