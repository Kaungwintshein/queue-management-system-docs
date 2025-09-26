# UI Implementation Guide v2.0

A comprehensive guide for implementing the enhanced frontend interfaces of the Queue Management System with authentication, multi-counter support, and advanced staff workflows.

## 🎯 Overview

The system consists of three distinct user interfaces with enhanced functionality:

1. **Multi-Counter Display Interface** - Customer-facing showcase display supporting multiple counters
2. **Staff Interface** - Enhanced queue management with authentication, counter selection, and advanced workflows
3. **Admin Interface** - System administration with authentication and comprehensive management tools

## 🛠️ Technology Stack

### Frontend Technologies

- **Framework**: Next.js 14+ with App Router and TypeScript
- **Styling**: Tailwind CSS v4 + shadcn/ui components
- **State Management**: React hooks with optimistic updates
- **Authentication**: Role-based authentication system
- **Real-time**: WebSocket integration for live updates
- **Icons**: Lucide React
- **Animations**: CSS transitions and transforms

### Key Dependencies

\`\`\`json
{
"next": "^14.0.0",
"react": "^18.0.0",
"typescript": "^5.0.0",
"tailwindcss": "^4.0.0",
"@radix-ui/react-\*": "latest",
"lucide-react": "^0.300.0",
"socket.io-client": "^4.0.0"
}
\`\`\`

## 🎨 Enhanced Design System

### Color Palette

\`\`\`css
/_ Updated color tokens for v2.0 _/
:root {
/_ Base colors _/
--background: #ffffff;
--foreground: #0f172a;

/_ Muted colors _/
--muted: #f8fafc;
--muted-foreground: #64748b;

/_ Border and input _/
--border: #e2e8f0;
--input: #e2e8f0;

/_ Primary brand _/
--primary: #0f172a;
--primary-foreground: #f8fafc;

/_ Secondary _/
--secondary: #f1f5f9;
--secondary-foreground: #0f172a;

/_ Accent colors _/
--accent: #f1f5f9;
--accent-foreground: #0f172a;

/_ Status colors _/
--success: #10b981;
--success-foreground: #ffffff;
--warning: #f59e0b;
--warning-foreground: #ffffff;
--destructive: #ef4444;
--destructive-foreground: #ffffff;

/_ Customer type indicators _/
--instant-customer: #3b82f6;
--browser-customer: #10b981;
--retail-customer: #8b5cf6;
}
\`\`\`

### Typography System

\`\`\`css
/_ Enhanced typography scale _/
.text-display {
font-size: 8rem;
line-height: 1;
font-weight: 700;
}

.text-counter {
font-size: 4rem;
line-height: 1.1;
font-weight: 600;
}

.text-token {
font-size: 2.5rem;
line-height: 1.2;
font-weight: 600;
}

.text-service-timer {
font-size: 1.5rem;
line-height: 1.4;
font-weight: 500;
color: var(--muted-foreground);
}
\`\`\`

## 🖥️ Enhanced Interface Specifications

### 1. Multi-Counter Display Interface

**Purpose**: Customer-facing display showing multiple counter queues simultaneously
**Target Screen**: Large display monitor (1920x1080 or larger)

#### Dynamic Layout Structure

\`\`\`
┌─────────────────────────────────────────────────────────────┐
│ QUEUE MANAGEMENT SYSTEM │
│ Active Counters: 4 | Total Waiting: 12 │
├─────────────────────────────────────────────────────────────┤
│ ┌─ Counter 1 ─────┐ ┌─ Counter 2 ─────┐ │
│ │ NOW SERVING │ │ NOW SERVING │ │
│ │ R-1003 │ │ B-2001 │ │
│ │ ┌─ Next ──────┐ │ │ ┌─ Next ──────┐ │ │
│ │ │ R-1004 │ │ │ │ B-2002 │ │ │
│ │ │ R-1005 │ │ │ │ B-2003 │ │ │
│ │ └─────────────┘ │ │ └─────────────┘ │ │
│ └─────────────────┘ └─────────────────┘ │
│ │
│ ┌─ Counter 3 ─────┐ ┌─ Counter 4 ─────┐ │
│ │ NOW SERVING │ │ NOW SERVING │ │
│ │ I-3001 │ │ I-4001 │ │
│ │ ┌─ Next ──────┐ │ │ ┌─ Next ──────┐ │ │
│ │ │ I-3002 │ │ │ │ I-4002 │ │ │
│ │ │ I-3003 │ │ │ │ I-4003 │ │ │
│ │ └─────────────┘ │ │ └─────────────┘ │ │
│ └─────────────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────┘
\`\`\`

#### Enhanced Data Schemas

\`\`\`typescript
// Enhanced schemas for multi-counter support
export const counterSchema = z.object({
id: z.string().uuid(),
name: z.string().min(1),
isActive: z.boolean(),
currentServing: tokenSchema.nullable(),
nextInQueue: z.array(tokenSchema),
staffId: z.string().uuid().optional(),
staffName: z.string().optional(),
});

export const multiCounterDisplaySchema = z.object({
counters: z.array(counterSchema),
globalStats: z.object({
totalActiveCounters: z.number(),
totalWaiting: z.number(),
totalServing: z.number(),
averageWaitTime: z.number(),
}),
});

export type Counter = z.infer<typeof counterSchema>;
export type MultiCounterDisplay = z.infer<typeof multiCounterDisplaySchema>;
\`\`\`

#### Multi-Counter Display Component

\`\`\`tsx
const MultiCounterDisplay: React.FC = () => {
const [displayData, setDisplayData] = useState<MultiCounterDisplay | null>(null);

// Real-time updates
useEffect(() => {
const socket = io(process.env.NEXT_PUBLIC_SOCKET_URL);

    socket.on('display:update', (data: MultiCounterDisplay) => {
      setDisplayData(data);
    });

    return () => socket.disconnect();

}, []);

if (!displayData) {
return <LoadingDisplay />;
}

const { counters, globalStats } = displayData;
const activeCounters = counters.filter(counter => counter.isActive);

return (

<div className="min-h-screen bg-background p-8">
{/_ Global Header _/}
<header className="text-center mb-12">
<h1 className="text-4xl font-light text-foreground mb-4">
Queue Management System
</h1>
<div className="flex justify-center gap-8 text-muted-foreground">
<span>Active Counters: {globalStats.totalActiveCounters}</span>
<span>Total Waiting: {globalStats.totalWaiting}</span>
<span>Currently Serving: {globalStats.totalServing}</span>
<span>Avg Wait: {globalStats.averageWaitTime}min</span>
</div>
</header>

      {/* Dynamic Counter Grid */}
      <div className={`grid gap-8 ${getGridClass(activeCounters.length)}`}>
        {activeCounters.map((counter) => (
          <CounterCard key={counter.id} counter={counter} />
        ))}
      </div>
    </div>

);
};

const CounterCard: React.FC<{ counter: Counter }> = ({ counter }) => {
return (

<div className="bg-card border border-border rounded-lg p-6">
{/_ Counter Header _/}
<div className="text-center mb-6">
<h2 className="text-2xl font-semibold text-foreground mb-2">
{counter.name}
</h2>
{counter.staffName && (
<p className="text-muted-foreground">Staff: {counter.staffName}</p>
)}
</div>

      {/* Now Serving */}
      <div className="text-center mb-8">
        <h3 className="text-xl text-muted-foreground mb-4">NOW SERVING</h3>
        <div className="text-counter text-foreground">
          {counter.currentServing?.number || '---'}
        </div>
        {counter.currentServing && (
          <div className="mt-4">
            <CustomerTypeBadge type={counter.currentServing.customerType} />
          </div>
        )}
      </div>

      {/* Next in Queue */}
      <div>
        <h4 className="text-lg font-medium text-foreground mb-4 text-center">
          Next in Queue
        </h4>
        <div className="space-y-2">
          {counter.nextInQueue.slice(0, 3).map((token) => (
            <div
              key={token.id}
              className="flex items-center justify-between p-3 bg-muted rounded-md"
            >
              <span className="font-medium">{token.number}</span>
              <CustomerTypeBadge type={token.customerType} size="sm" />
            </div>
          ))}
          {counter.nextInQueue.length === 0 && (
            <p className="text-center text-muted-foreground py-4">
              No customers waiting
            </p>
          )}
        </div>
      </div>
    </div>

);
};

// Helper function for responsive grid
const getGridClass = (count: number): string => {
switch (count) {
case 1: return 'grid-cols-1 max-w-2xl mx-auto';
case 2: return 'grid-cols-1 lg:grid-cols-2 max-w-6xl mx-auto';
case 3: return 'grid-cols-1 lg:grid-cols-2 xl:grid-cols-3';
case 4: return 'grid-cols-1 lg:grid-cols-2 xl:grid-cols-4';
default: return 'grid-cols-1 lg:grid-cols-2 xl:grid-cols-3 2xl:grid-cols-4';
}
};
\`\`\`

### 2. Enhanced Staff Interface with Authentication

**Purpose**: Comprehensive queue management with role-based access
**Target Device**: Tablet or desktop

#### Authentication System

\`\`\`typescript
// Authentication schemas
export const staffAuthSchema = z.object({
username: z.string().min(3),
password: z.string().min(6),
role: z.enum(['staff', 'admin']),
});

export const staffUserSchema = z.object({
id: z.string().uuid(),
username: z.string(),
role: z.enum(['staff', 'admin']),
assignedCounters: z.array(z.string()),
isActive: z.boolean(),
});

export type StaffAuth = z.infer<typeof staffAuthSchema>;
export type StaffUser = z.infer<typeof staffUserSchema>;
\`\`\`

#### Authentication Wrapper Component

\`\`\`tsx
const AuthWrapper: React.FC<{
children: (user: StaffUser) => React.ReactNode;
requiredRole?: 'staff' | 'admin';
}> = ({ children, requiredRole = 'staff' }) => {
const [user, setUser] = useState<StaffUser | null>(null);
const [isLoading, setIsLoading] = useState(true);

useEffect(() => {
// Check for existing session
const checkAuth = async () => {
try {
const token = localStorage.getItem('staff_token');
if (token) {
const userData = await verifyToken(token);
setUser(userData);
}
} catch (error) {
localStorage.removeItem('staff_token');
} finally {
setIsLoading(false);
}
};

    checkAuth();

}, []);

if (isLoading) {
return <LoadingSpinner />;
}

if (!user || (requiredRole === 'admin' && user.role !== 'admin')) {
return <LoginForm onLogin={setUser} />;
}

return <>{children(user)}</>;
};
\`\`\`

#### Counter Selection Interface

\`\`\`tsx
const CounterSelection: React.FC<{
user: StaffUser;
onCounterSelect: (counterId: string) => void;
}> = ({ user, onCounterSelect }) => {
const [availableCounters, setAvailableCounters] = useState<Counter[]>([]);
const [selectedCounter, setSelectedCounter] = useState<string>('');

return (

<div className="min-h-screen bg-background flex items-center justify-center p-8">
<div className="w-full max-w-md">
<div className="text-center mb-8">
<h1 className="text-3xl font-bold text-foreground mb-2">
Welcome, {user.username}
</h1>
<p className="text-muted-foreground">Select your counter to begin</p>
</div>

        <div className="bg-card border border-border rounded-lg p-6">
          <label className="block text-sm font-medium text-foreground mb-4">
            Counter
          </label>

          <select
            value={selectedCounter}
            onChange={(e) => setSelectedCounter(e.target.value)}
            className="w-full p-3 border border-input rounded-md bg-background text-foreground"
          >
            <option value="">Choose Your Counter</option>
            {availableCounters.map((counter) => (
              <option key={counter.id} value={counter.id}>
                {counter.name}
              </option>
            ))}
          </select>

          <div className="flex gap-4 mt-6">
            <button
              onClick={() => window.location.reload()}
              className="flex-1 px-4 py-2 bg-destructive text-destructive-foreground rounded-md hover:bg-destructive/90"
            >
              Go Back ✕
            </button>
            <button
              onClick={() => selectedCounter && onCounterSelect(selectedCounter)}
              disabled={!selectedCounter}
              className="flex-1 px-4 py-2 bg-primary text-primary-foreground rounded-md hover:bg-primary/90 disabled:opacity-50"
            >
              Submit ▶
            </button>
          </div>
        </div>
      </div>
    </div>

);
};
\`\`\`

#### Enhanced Staff Service Interface

\`\`\`tsx
const StaffServiceInterface: React.FC<{
user: StaffUser;
counterId: string;
}> = ({ user, counterId }) => {
const [currentToken, setCurrentToken] = useState<Token | null>(null);
const [nextInQueue, setNextInQueue] = useState<Token[]>([]);
const [noShowQueue, setNoShowQueue] = useState<Token[]>([]);
const [serviceTimer, setServiceTimer] = useState<number>(0);

// Service timer effect
useEffect(() => {
if (currentToken?.calledAt) {
const startTime = new Date(currentToken.calledAt).getTime();
const interval = setInterval(() => {
setServiceTimer(Math.floor((Date.now() - startTime) / 1000));
}, 1000);
return () => clearInterval(interval);
}
}, [currentToken]);

const formatTime = (seconds: number): string => {
const mins = Math.floor(seconds / 60);
const secs = seconds % 60;
return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
};

const handleServed = async () => {
if (currentToken) {
await completeService(currentToken.id, user.id);
setCurrentToken(null);
setServiceTimer(0);
}
};

const handleRecall = async () => {
if (currentToken) {
await recallToken(currentToken.id);
}
};

const handleNoShow = async () => {
if (currentToken) {
await markNoShow(currentToken.id);
setNoShowQueue(prev => [...prev, currentToken]);
setCurrentToken(null);
setServiceTimer(0);
}
};

const handleCallNext = async () => {
if (nextInQueue.length > 0) {
const nextToken = nextInQueue[0];
await callToken(nextToken.id, counterId);
setCurrentToken(nextToken);
setNextInQueue(prev => prev.slice(1));
setServiceTimer(0);
}
};

const handleReturnCall = async (tokenId: string) => {
const token = noShowQueue.find(t => t.id === tokenId);
if (token) {
await recallToken(tokenId);
setNoShowQueue(prev => prev.filter(t => t.id !== tokenId));
setNextInQueue(prev => [token, ...prev]);
}
};

return (

<div className="min-h-screen bg-background">
{/_ Header _/}
<header className="bg-card border-b border-border p-4">
<div className="flex items-center justify-between">
<h1 className="text-xl font-semibold text-foreground">
Counter {counterId} - {user.username}
</h1>
<button
onClick={() => logout()}
className="px-4 py-2 text-muted-foreground hover:text-foreground" >
Logout
</button>
</div>
</header>

      <div className="p-8">
        {/* Current Service Panel */}
        {currentToken ? (
          <div className="bg-card border border-border rounded-lg p-8 mb-8 text-center">
            <div className="text-display text-foreground mb-4">
              {currentToken.number}
            </div>
            <div className="text-service-timer mb-8">
              {formatTime(serviceTimer)}
            </div>

            {/* Action Buttons */}
            <div className="grid grid-cols-2 gap-4 max-w-md mx-auto">
              <button
                onClick={handleServed}
                className="px-6 py-3 bg-success text-success-foreground rounded-md hover:bg-success/90 font-medium"
              >
                Served ▶
              </button>
              <button
                onClick={handleRecall}
                className="px-6 py-3 bg-success text-success-foreground rounded-md hover:bg-success/90 font-medium"
              >
                Recall ▶
              </button>
              <button
                onClick={handleNoShow}
                className="px-6 py-3 bg-success text-success-foreground rounded-md hover:bg-success/90 font-medium"
              >
                No Show ▶
              </button>
              <button
                onClick={handleCallNext}
                disabled={nextInQueue.length === 0}
                className="px-6 py-3 bg-muted text-muted-foreground rounded-md hover:bg-muted/90 font-medium disabled:opacity-50"
              >
                Call Next ▶
              </button>
            </div>

            <div className="mt-6 text-sm text-muted-foreground">
              Service: Registration | Counter: Counter {counterId} | 🖊️
            </div>
          </div>
        ) : (
          <div className="bg-card border border-border rounded-lg p-8 mb-8 text-center">
            <p className="text-muted-foreground mb-4">No customer currently being served</p>
            <button
              onClick={handleCallNext}
              disabled={nextInQueue.length === 0}
              className="px-6 py-3 bg-primary text-primary-foreground rounded-md hover:bg-primary/90 disabled:opacity-50"
            >
              Call Next Customer
            </button>
          </div>
        )}

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
          {/* Queue Panel */}
          <div className="bg-card border border-border rounded-lg p-6">
            <h2 className="text-xl font-semibold text-foreground mb-4">
              Queue ({nextInQueue.length})
            </h2>
            <div className="space-y-3">
              {nextInQueue.map((token) => (
                <div
                  key={token.id}
                  className="flex items-center justify-between p-3 bg-muted rounded-md"
                >
                  <div>
                    <span className="font-medium text-foreground">{token.number}</span>
                    <span className="ml-2 text-sm text-muted-foreground">
                      {token.customerType}
                    </span>
                  </div>
                  <CustomerTypeBadge type={token.customerType} size="sm" />
                </div>
              ))}
              {nextInQueue.length === 0 && (
                <p className="text-center text-muted-foreground py-8">
                  No customers in queue
                </p>
              )}
            </div>
          </div>

          {/* No Show Panel */}
          <div className="bg-card border border-border rounded-lg p-6">
            <h2 className="text-xl font-semibold text-foreground mb-4">
              No Show ({noShowQueue.length})
            </h2>
            <div className="space-y-3">
              {noShowQueue.map((token) => (
                <div
                  key={token.id}
                  className="flex items-center justify-between p-3 bg-muted rounded-md"
                >
                  <div className="flex items-center gap-3">
                    <button
                      onClick={() => handleReturnCall(token.id)}
                      className="w-8 h-8 bg-success text-success-foreground rounded-full flex items-center justify-center hover:bg-success/90"
                      title="Return Call"
                    >
                      ↻
                    </button>
                    <span className="font-medium text-foreground">{token.number}</span>
                  </div>
                  <span className="px-2 py-1 bg-destructive text-destructive-foreground text-xs rounded">
                    N
                  </span>
                </div>
              ))}
              {noShowQueue.length === 0 && (
                <p className="text-center text-muted-foreground py-8">
                  No no-show customers
                </p>
              )}
            </div>
          </div>
        </div>
      </div>
    </div>

);
};
\`\`\`

### 3. Enhanced Admin Interface

**Purpose**: Comprehensive system administration with authentication
**Target Device**: Desktop/laptop

#### Admin Dashboard Component

\`\`\`tsx
const AdminInterface: React.FC<{ user: StaffUser }> = ({ user }) => {
const [activeView, setActiveView] = useState<'dashboard' | 'queue' | 'staff' | 'analytics'>('dashboard');

return (

<div className="min-h-screen bg-background">
<AdminHeader user={user} />

      <div className="flex">
        <AdminSidebar activeView={activeView} onViewChange={setActiveView} />

        <main className="flex-1 p-8">
          {activeView === 'dashboard' && <AdminDashboard />}
          {activeView === 'queue' && <QueueManagement />}
          {activeView === 'staff' && <StaffManagement />}
          {activeView === 'analytics' && <Analytics />}
        </main>
      </div>
    </div>

);
};
\`\`\`

## 🔄 Real-time Updates & WebSocket Integration

### Enhanced WebSocket Implementation

\`\`\`typescript
// Enhanced real-time updates for multi-counter system
export const useRealTimeUpdates = () => {
const [socket, setSocket] = useState<Socket | null>(null);

useEffect(() => {
const newSocket = io(process.env.NEXT_PUBLIC_SOCKET_URL);
setSocket(newSocket);

    // Multi-counter display updates
    newSocket.on('display:counters-updated', (data: MultiCounterDisplay) => {
      // Update display interface
    });

    // Staff interface updates
    newSocket.on('staff:queue-updated', (data: { counterId: string; queue: Token[] }) => {
      // Update specific counter queue
    });

    // Token status changes
    newSocket.on('token:status-changed', (data: { token: Token; action: string }) => {
      // Handle token status updates
    });

    return () => newSocket.disconnect();

}, []);

return socket;
};
\`\`\`

## 📱 Responsive Design Enhancements

### Enhanced Breakpoints

\`\`\`css
/_ Enhanced responsive design _/
@media (max-width: 768px) {
.text-display { font-size: 4rem; }
.text-counter { font-size: 2.5rem; }
}

@media (min-width: 1920px) {
.text-display { font-size: 10rem; }
.text-counter { font-size: 5rem; }
}
\`\`\`

## 🔐 Security Considerations

### Authentication Flow

1. **Login**: Username/password validation with JWT tokens
2. **Session Management**: Secure token storage and refresh
3. **Role-based Access**: Staff vs Admin permissions
4. **Counter Assignment**: Restrict staff to assigned counters

### Data Validation

- All API responses validated with Zod schemas
- Form inputs sanitized and validated
- Real-time data integrity checks

## 🧪 Testing Strategy

### Component Testing

\`\`\`typescript
// Enhanced testing for authentication and multi-counter features
describe('StaffInterface', () => {
it('requires authentication', () => {
render(<StaffInterface />);
expect(screen.getByText('Login')).toBeInTheDocument();
});

it('shows counter selection after login', async () => {
const mockUser = { id: '1', username: 'staff1', role: 'staff' as const };
render(<StaffInterface />);

    // Mock login
    fireEvent.click(screen.getByText('Login'));

    await waitFor(() => {
      expect(screen.getByText('Choose Your Counter')).toBeInTheDocument();
    });

});
});
\`\`\`

## 🚀 Deployment & Performance

### Build Optimization

\`\`\`json
{
"scripts": {
"build:display": "NEXT_PUBLIC_INTERFACE=display next build",
"build:staff": "NEXT_PUBLIC_INTERFACE=staff next build",
"build:admin": "NEXT_PUBLIC_INTERFACE=admin next build"
}
}
\`\`\`

### Performance Considerations

- **Code Splitting**: Interface-specific bundles
- **Real-time Optimization**: Efficient WebSocket event handling
- **Caching**: Optimistic updates with cache invalidation
- **Responsive Images**: Optimized assets for different screen sizes

## 📋 Implementation Checklist

### Phase 1: Core Enhancements

- [x] Multi-counter display interface
- [x] Authentication system
- [x] Counter selection workflow
- [x] Enhanced staff interface with four actions
- [x] No-show queue management

### Phase 2: Advanced Features

- [ ] Advanced analytics dashboard
- [ ] Staff performance metrics
- [ ] Customer feedback system
- [ ] Mobile app integration

### Phase 3: Optimization

- [ ] Performance monitoring
- [ ] A/B testing framework
- [ ] Advanced caching strategies
- [ ] Offline support

---

_This enhanced guide provides the foundation for building a comprehensive, secure, and user-friendly queue management system with advanced multi-counter support and role-based authentication._
