# UI Documentation

This directory contains all UI-related documentation for the Queue Management System frontend.

## 📋 Available Documentation

### Implementation Guides

- **[UI Implementation Guide v1](./ui-implementation-guide.md)** - Initial UI implementation guide
- **[UI Implementation Guide v2](./ui-implementation-guide-v2.md)** - Updated UI implementation guide with improvements
- **[UI Implementation Guide v3](./ui-implementation-guide-v3.md)** - Enhanced UI implementation guide with advanced features
- **[UI Implementation Guide v4](./ui-implementation-guide-v4.md)** - **Latest** Modern glass morphism design with contemporary styling

## 🎨 UI Architecture

The frontend is built with modern web technologies:

- **Framework**: Next.js 14 with App Router
- **Styling**: TailwindCSS with shadcn/ui components
- **State Management**: Zustand for global state
- **Data Fetching**: TanStack Query (React Query)
- **Authentication**: JWT-based with role-based access control
- **Real-time**: Socket.io client for live updates

## 🏗️ Component Structure

```
ui/
├── app/                    # Next.js App Router pages
│   ├── admin/             # Admin portal pages
│   ├── display/           # Display interface
│   ├── staff/             # Staff interface
│   └── login/             # Authentication pages
├── components/            # Reusable React components
│   ├── admin/             # Admin-specific components
│   ├── ui/                # Base UI components (shadcn/ui)
│   └── ...                # Other feature components
├── lib/                   # Utility libraries
│   ├── api/               # API client and hooks
│   ├── stores/            # Zustand stores
│   └── types/             # TypeScript type definitions
└── styles/                # Global styles and CSS
```

## 🚀 Key Features

### 🎨 Modern Design System

- **Glass Morphism**: Contemporary UI with semi-transparent elements and backdrop blur
- **Gradient Accents**: Beautiful gradient backgrounds and overlays
- **Smooth Animations**: Hover effects, transitions, and micro-interactions
- **Responsive Design**: Mobile-first approach with adaptive layouts
- **Color-Coded System**: Intuitive color coding for different queue states

### 🔐 Authentication & Authorization

- **Login/Logout**: Secure authentication with JWT tokens
- **Role-based Access**: Different interfaces for admin, staff, and display
- **Protected Routes**: Automatic redirection based on authentication status
- **Persistent Sessions**: State persistence across browser sessions

### 🏢 Admin Portal

- **User Management**: CRUD operations for users with advanced features
- **Counter Management**: Manage service counters and staff assignments
- **Role Management**: Handle user roles and permissions
- **Token Configuration**: Manage service types and priority settings
- **Modern Data Tables**: Advanced tables with glass morphism styling
- **Responsive Design**: Works on desktop and mobile devices

### 👨‍💼 Staff Interface

- **Counter Assignment**: Staff can be assigned to specific counters
- **Service Type Selection**: Choose which service types to prioritize
- **Advanced Queue Management**: Individual token calling and priority filtering
- **Real-time Updates**: Live updates via WebSocket connections
- **Modern Dashboard**: Glass morphism cards with gradient headers

### 📺 Display Interface

- **Queue Display**: Show current queue status and next numbers
- **Counter Status**: Display which counters are active
- **Real-time Updates**: Live queue updates without page refresh
- **Voice Announcements**: Audio notifications for called tokens
- **Modern Styling**: Clean, contemporary customer-facing display

### 🎫 Token Management

- **Service Type Configuration**: Customizable service types with priority settings
- **Queue Prioritization**: Staff can select which service types to prioritize
- **Individual Token Calling**: Call specific tokens by ID
- **No-Show Management**: Handle skipped tokens with recall functionality
- **Service Completion Tracking**: Monitor service duration and completion rates

## 🎯 UI Components

### Base Components (shadcn/ui)

- **Button**: Various button styles and sizes
- **Input**: Form input components with validation
- **Table**: Data table with sorting and filtering
- **Dialog**: Modal dialogs for forms and confirmations
- **Card**: Content containers with consistent styling
- **Badge**: Status indicators and labels

### Custom Components

- **LoginForm**: Authentication form with validation
- **UserManagementTable**: Advanced user management interface
- **CounterManagementTable**: Counter management with staff assignment
- **RoleManagementTable**: Role management interface
- **AdminLayout**: Responsive admin portal layout with sidebar

## 🔧 Development Setup

1. **Install Dependencies**:

   ```bash
   cd ui
   npm install
   ```

2. **Environment Setup**:

   ```bash
   cp .env.example .env.local
   # Configure your environment variables
   ```

3. **Start Development Server**:

   ```bash
   npm run dev
   ```

4. **Build for Production**:
   ```bash
   npm run build
   npm start
   ```

## 🎨 Styling Guidelines

### TailwindCSS Configuration

- **Custom Colors**: Brand colors and semantic color system
- **Responsive Design**: Mobile-first approach with breakpoints
- **Dark Mode**: Built-in dark/light theme support
- **Component Variants**: Consistent component styling

### Design System

- **Typography**: Consistent font sizes and weights
- **Spacing**: Standardized spacing scale
- **Colors**: Semantic color palette for different states
- **Components**: Reusable component library

## 🔄 State Management

### Zustand Stores

- **AuthStore**: Authentication state and user information
- **QueueStore**: Queue and counter state management

### React Query

- **API Caching**: Automatic caching of API responses
- **Background Updates**: Automatic data refetching
- **Optimistic Updates**: Immediate UI updates with rollback
- **Error Handling**: Centralized error handling and retry logic

## 📱 Responsive Design

- **Mobile First**: Designed for mobile devices first
- **Breakpoints**:
  - `sm`: 640px and up
  - `md`: 768px and up
  - `lg`: 1024px and up
  - `xl`: 1280px and up
- **Touch Friendly**: Large touch targets and gestures
- **Adaptive Layout**: Components adapt to screen size

## 🧪 Testing

- **Component Testing**: Unit tests for individual components
- **Integration Testing**: Tests for component interactions
- **E2E Testing**: Full user workflow testing
- **Visual Testing**: Screenshot comparisons for UI consistency

## 🚀 Performance

- **Code Splitting**: Automatic code splitting with Next.js
- **Image Optimization**: Next.js Image component for optimized images
- **Bundle Analysis**: Webpack bundle analyzer for optimization
- **Lazy Loading**: Components loaded on demand
- **Caching**: Aggressive caching strategies for better performance
