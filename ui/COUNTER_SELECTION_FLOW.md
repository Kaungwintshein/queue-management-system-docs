# Counter Selection Flow Documentation

## Overview

The counter selection flow is designed for staff members to select their assigned counter after logging in. This ensures that only one staff member can be assigned to each counter at a time.

## User Flow

### 1. Staff Login

- Staff member logs in with their credentials
- System validates credentials and returns user information
- If user role is "staff", redirect to counter selection page

### 2. Counter Selection Page

- **Route**: `/counter-selection`
- **Component**: `CounterSelection`
- **Purpose**: Allow staff to select an available counter

#### Features:

- **Available Counters List**: Shows all active counters that are not assigned to any staff
- **Counter Selection**: Dropdown to select from available counters
- **Submit Button**: Confirms counter selection
- **Go Back Button**: Returns to previous page
- **Loading States**: Shows loading indicators during API calls
- **Error Handling**: Displays appropriate error messages

### 3. Counter Assignment

- Staff selects a counter from the dropdown
- System validates that:
  - Counter exists and is active
  - Counter is not already assigned
  - Staff doesn't already have a counter assigned
- If valid, counter is assigned to the staff member
- Success message is displayed
- Staff is redirected to staff dashboard

### 4. Staff Dashboard

- **Route**: `/staff`
- **Component**: `StaffDashboardPage`
- **Purpose**: Main interface for staff operations

#### Features:

- **Counter Status**: Shows currently assigned counter
- **Release Counter**: Option to release current counter assignment
- **Select New Counter**: Option to change counter assignment
- **Queue Management**: Placeholder for future queue operations
- **Quick Actions**: Refresh status, change counter, view statistics

## API Endpoints Used

### 1. Get Available Counters

```
GET /api/counters/available
```

- **Purpose**: Fetch counters available for selection
- **Auth**: Staff only
- **Response**: Array of available counters

### 2. Select Counter

```
POST /api/counters/select
```

- **Purpose**: Assign selected counter to staff
- **Auth**: Staff only
- **Body**: `{ "counterId": "uuid" }`
- **Response**: Updated counter with staff assignment

### 3. Release Counter

```
POST /api/counters/release
```

- **Purpose**: Release current counter assignment
- **Auth**: Staff only
- **Response**: Success message

### 4. Get Counters (for dashboard)

```
GET /api/counters?assigned=true
```

- **Purpose**: Get counters with assignments for dashboard
- **Auth**: Staff only
- **Response**: Array of counters with assignment details

## Component Structure

### CounterSelection Component

```typescript
interface CounterSelectionProps {
  onCounterSelected?: (counter: Counter) => void;
  onGoBack?: () => void;
}
```

**Props:**

- `onCounterSelected`: Optional callback when counter is selected
- `onGoBack`: Optional callback for go back action

**State:**

- `counters`: Array of available counters
- `selectedCounterId`: Currently selected counter ID
- `loading`: Loading state for API calls
- `submitting`: Loading state for form submission

### StaffDashboardPage Component

**State:**

- `assignedCounter`: Currently assigned counter
- `loading`: Loading state for API calls

## Error Handling

### Common Error Scenarios:

1. **No Available Counters**: Shows message to contact administrator
2. **Counter Already Assigned**: Shows error message
3. **Staff Already Has Counter**: Shows error message
4. **Network Errors**: Shows generic error message
5. **Invalid Counter ID**: Shows validation error

### Error Messages:

- "Please select a counter"
- "Counter not found or already assigned"
- "You already have a counter assigned"
- "Failed to load available counters"
- "Failed to select counter"
- "No counters are currently available. Please contact your administrator."

## Security Considerations

1. **Authentication Required**: All endpoints require valid JWT token
2. **Role-Based Access**: Only staff members can access counter selection
3. **Organization Isolation**: Staff can only see counters from their organization
4. **One Counter Per Staff**: System prevents multiple counter assignments
5. **Audit Logging**: All counter assignments are logged for tracking

## UI/UX Features

### Visual Design:

- **Gradient Background**: Blue gradient for professional look
- **Card Layout**: Clean card-based design
- **Icons**: Monitor icons for counter representation
- **Loading States**: Spinner animations during API calls
- **Responsive Design**: Works on desktop and mobile

### User Experience:

- **Clear Instructions**: "Choose Your Counter" heading
- **Intuitive Controls**: Standard dropdown and button patterns
- **Feedback**: Toast notifications for success/error states
- **Navigation**: Clear go back and submit options
- **Status Display**: Shows current user information

## Future Enhancements

1. **Real-time Updates**: WebSocket integration for live counter status
2. **Counter Statistics**: Show counter usage statistics
3. **Queue Integration**: Direct queue management from counter selection
4. **Multi-language Support**: Internationalization
5. **Accessibility**: Enhanced screen reader support
6. **Offline Support**: PWA capabilities for offline counter selection

## Testing Scenarios

### Happy Path:

1. Staff logs in successfully
2. Redirected to counter selection
3. Sees available counters
4. Selects a counter
5. Counter is assigned successfully
6. Redirected to staff dashboard
7. Can see assigned counter status

### Error Scenarios:

1. No available counters
2. Counter selection fails
3. Network connectivity issues
4. Invalid authentication
5. Counter already assigned by another staff

### Edge Cases:

1. Multiple staff trying to select same counter
2. Counter becomes unavailable during selection
3. Staff loses authentication during process
4. Browser refresh during counter selection
