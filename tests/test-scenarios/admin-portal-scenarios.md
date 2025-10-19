# Admin Portal Test Scenarios

This document outlines comprehensive test scenarios for the Admin Portal functionality using Appium for cross-platform testing.

## 📋 Table of Contents

1. [User Management Scenarios](#user-management-scenarios)
2. [Counter Management Scenarios](#counter-management-scenarios)
3. [Role Management Scenarios](#role-management-scenarios)
4. [Dashboard Scenarios](#dashboard-scenarios)
5. [Security Scenarios](#security-scenarios)
6. [Performance Scenarios](#performance-scenarios)

## 👥 User Management Scenarios

### Scenario 1: Complete User Lifecycle

**Objective**: Test the complete user lifecycle from creation to deletion

**Test Steps**:

1. **Login as Admin**

   ```javascript
   // Navigate to login page
   await driver.$("#login-button").click();

   // Enter admin credentials
   await driver.$("#username").setValue("admin");
   await driver.$("#password").setValue("admin123");

   // Submit login
   await driver.$("#submit-login").click();

   // Verify dashboard access
   await expect(driver.$("#admin-dashboard")).toBeDisplayed();
   ```

2. **Create New Staff User**

   ```javascript
   // Navigate to user management
   await driver.$("#user-management-tab").click();

   // Click create user button
   await driver.$("#create-user-button").click();

   // Fill user form
   await driver.$("#new-username").setValue("newstaff");
   await driver.$("#new-email").setValue("newstaff@test.com");
   await driver.$("#new-password").setValue("staff123");
   await driver.$("#role-select").click();
   await driver.$("#role-staff").click();

   // Submit user creation
   await driver.$("#submit-user").click();

   // Verify success message
   await expect(driver.$("#success-message")).toBeDisplayed();
   ```

3. **Verify User in List**

   ```javascript
   // Check user appears in list
   const userList = await driver.$("#user-list");
   await expect(userList).toBeDisplayed();

   // Verify new user details
   const newUser = await driver.$("#user-newstaff");
   await expect(newUser).toBeDisplayed();
   ```

4. **Update User Information**

   ```javascript
   // Click on user to edit
   await driver.$("#user-newstaff").click();

   // Update email
   await driver.$("#edit-email").clear();
   await driver.$("#edit-email").setValue("updated@test.com");

   // Save changes
   await driver.$("#save-user").click();

   // Verify update success
   await expect(driver.$("#update-success")).toBeDisplayed();
   ```

5. **Manage User Status**

   ```javascript
   // Ban user
   await driver.$("#ban-user-button").click();
   await driver.$("#ban-reason").setValue("Test ban");
   await driver.$("#confirm-ban").click();

   // Verify user is banned
   await expect(driver.$("#user-status-banned")).toBeDisplayed();

   // Reactivate user
   await driver.$("#reactivate-user-button").click();
   await driver.$("#confirm-reactivate").click();

   // Verify user is active
   await expect(driver.$("#user-status-active")).toBeDisplayed();
   ```

6. **Reset User Password**

   ```javascript
   // Click reset password
   await driver.$("#reset-password-button").click();

   // Confirm reset
   await driver.$("#confirm-reset").click();

   // Verify temporary password generated
   await expect(driver.$("#temporary-password")).toBeDisplayed();
   ```

7. **Delete User**

   ```javascript
   // Click delete user
   await driver.$("#delete-user-button").click();

   // Confirm deletion
   await driver.$("#confirm-delete").click();

   // Verify user removed from list
   await expect(driver.$("#user-newstaff")).not.toBeDisplayed();
   ```

**Expected Results**:

- User creation successful with proper validation
- User appears in management list
- User updates persist correctly
- Status changes work as expected
- Password reset generates temporary password
- User deletion removes user completely

### Scenario 2: Bulk User Operations

**Objective**: Test bulk operations on multiple users

**Test Steps**:

1. **Select Multiple Users**

   ```javascript
   // Select multiple users
   await driver.$("#user-checkbox-1").click();
   await driver.$("#user-checkbox-2").click();
   await driver.$("#user-checkbox-3").click();

   // Verify selection count
   const selectedCount = await driver.$("#selected-count");
   await expect(selectedCount).toHaveText("3 users selected");
   ```

2. **Bulk Status Change**

   ```javascript
   // Click bulk actions
   await driver.$("#bulk-actions").click();

   // Select bulk ban
   await driver.$("#bulk-ban").click();

   // Enter reason
   await driver.$("#bulk-ban-reason").setValue("Bulk test ban");

   // Confirm bulk action
   await driver.$("#confirm-bulk-action").click();

   // Verify all selected users are banned
   const bannedUsers = await driver.$$(".user-status-banned");
   await expect(bannedUsers.length).toBe(3);
   ```

3. **Bulk Role Change**

   ```javascript
   // Select different users
   await driver.$("#user-checkbox-4").click();
   await driver.$("#user-checkbox-5").click();

   // Bulk role change
   await driver.$("#bulk-role-change").click();
   await driver.$("#new-role-select").click();
   await driver.$("#role-staff").click();

   // Confirm role change
   await driver.$("#confirm-role-change").click();

   // Verify role changes
   const staffUsers = await driver.$$(".user-role-staff");
   await expect(staffUsers.length).toBeGreaterThan(0);
   ```

**Expected Results**:

- Multiple users can be selected
- Bulk operations affect all selected users
- Status changes apply correctly
- Role changes persist

## 🏢 Counter Management Scenarios

### Scenario 3: Complete Counter Lifecycle

**Objective**: Test counter management from creation to deletion

**Test Steps**:

1. **Create New Counter**

   ```javascript
   // Navigate to counter management
   await driver.$("#counter-management-tab").click();

   // Click create counter
   await driver.$("#create-counter-button").click();

   // Fill counter form
   await driver.$("#counter-name").setValue("New Counter");
   await driver
     .$("#counter-description")
     .setValue("Test counter for queue management");
   await driver.$("#counter-active").click();

   // Submit counter creation
   await driver.$("#submit-counter").click();

   // Verify success
   await expect(driver.$("#counter-created-success")).toBeDisplayed();
   ```

2. **Assign Staff to Counter**

   ```javascript
   // Click on created counter
   await driver.$("#counter-New-Counter").click();

   // Click assign staff
   await driver.$("#assign-staff-button").click();

   // Select staff member
   await driver.$("#staff-select").click();
   await driver.$("#staff-member-1").click();

   // Confirm assignment
   await driver.$("#confirm-assignment").click();

   // Verify assignment
   await expect(driver.$("#assigned-staff")).toHaveText("Staff Member 1");
   ```

3. **Update Counter Information**

   ```javascript
   // Click edit counter
   await driver.$("#edit-counter-button").click();

   // Update counter name
   await driver.$("#edit-counter-name").clear();
   await driver.$("#edit-counter-name").setValue("Updated Counter Name");

   // Update description
   await driver.$("#edit-counter-description").clear();
   await driver.$("#edit-counter-description").setValue("Updated description");

   // Save changes
   await driver.$("#save-counter").click();

   // Verify updates
   await expect(driver.$("#counter-Updated-Counter-Name")).toBeDisplayed();
   ```

4. **Manage Counter Status**

   ```javascript
   // Toggle counter status
   await driver.$("#counter-status-toggle").click();

   // Verify status change
   await expect(driver.$("#counter-status-inactive")).toBeDisplayed();

   // Reactivate counter
   await driver.$("#counter-status-toggle").click();
   await expect(driver.$("#counter-status-active")).toBeDisplayed();
   ```

5. **Unassign Staff**

   ```javascript
   // Click unassign staff
   await driver.$("#unassign-staff-button").click();

   // Confirm unassignment
   await driver.$("#confirm-unassign").click();

   // Verify unassignment
   await expect(driver.$("#assigned-staff")).toHaveText("No staff assigned");
   ```

6. **Delete Counter**

   ```javascript
   // Click delete counter
   await driver.$("#delete-counter-button").click();

   // Confirm deletion
   await driver.$("#confirm-delete-counter").click();

   // Verify counter removed
   await expect(driver.$("#counter-Updated-Counter-Name")).not.toBeDisplayed();
   ```

**Expected Results**:

- Counter creation successful
- Staff assignment works correctly
- Counter updates persist
- Status changes affect queue behavior
- Unassignment removes staff properly
- Counter deletion removes completely

### Scenario 4: Counter Assignment Conflicts

**Objective**: Test handling of counter assignment conflicts

**Test Steps**:

1. **Assign Staff to Multiple Counters**

   ```javascript
   // Assign staff to first counter
   await driver.$("#counter-1").click();
   await driver.$("#assign-staff-button").click();
   await driver.$("#staff-member-1").click();
   await driver.$("#confirm-assignment").click();

   // Try to assign same staff to second counter
   await driver.$("#counter-2").click();
   await driver.$("#assign-staff-button").click();
   await driver.$("#staff-member-1").click();

   // Verify conflict warning
   await expect(driver.$("#assignment-conflict-warning")).toBeDisplayed();
   ```

2. **Resolve Assignment Conflict**

   ```javascript
   // Choose to reassign
   await driver.$("#reassign-option").click();

   // Confirm reassignment
   await driver.$("#confirm-reassignment").click();

   // Verify staff moved to new counter
   await expect(driver.$("#counter-2 .assigned-staff")).toHaveText(
     "Staff Member 1"
   );
   await expect(driver.$("#counter-1 .assigned-staff")).toHaveText(
     "No staff assigned"
   );
   ```

**Expected Results**:

- Conflict detection works correctly
- Warning messages are clear
- Reassignment resolves conflicts
- Previous assignments are updated

## 🔐 Role Management Scenarios

### Scenario 5: Role Permission Management

**Objective**: Test role-based permission management

**Test Steps**:

1. **View Role Information**

   ```javascript
   // Navigate to role management
   await driver.$("#role-management-tab").click();

   // View role details
   await driver.$("#role-staff").click();

   // Verify permissions displayed
   await expect(driver.$("#role-permissions")).toBeDisplayed();
   await expect(driver.$("#permission-counter-read")).toBeDisplayed();
   await expect(driver.$("#permission-user-create")).not.toBeDisplayed();
   ```

2. **Change User Role**

   ```javascript
   // Navigate to user management
   await driver.$("#user-management-tab").click();

   // Select user
   await driver.$("#user-testuser").click();

   // Change role
   await driver.$("#change-role-button").click();
   await driver.$("#new-role-select").click();
   await driver.$("#role-admin").click();

   // Confirm role change
   await driver.$("#confirm-role-change").click();

   // Verify role change
   await expect(driver.$("#user-role-admin")).toBeDisplayed();
   ```

3. **Test Permission Changes**

   ```javascript
   // Login as user with new role
   await driver.$("#logout-button").click();
   await driver.$("#username").setValue("testuser");
   await driver.$("#password").setValue("testpass");
   await driver.$("#submit-login").click();

   // Verify admin permissions
   await expect(driver.$("#user-management-tab")).toBeDisplayed();
   await expect(driver.$("#counter-management-tab")).toBeDisplayed();
   ```

**Expected Results**:

- Role information displays correctly
- Role changes apply immediately
- Permissions update based on role
- Access control works properly

## 📊 Dashboard Scenarios

### Scenario 6: Dashboard Data Display

**Objective**: Test dashboard data accuracy and real-time updates

**Test Steps**:

1. **View Dashboard Metrics**

   ```javascript
   // Navigate to dashboard
   await driver.$("#dashboard-tab").click();

   // Verify key metrics
   await expect(driver.$("#total-users")).toBeDisplayed();
   await expect(driver.$("#active-counters")).toBeDisplayed();
   await expect(driver.$("#queue-length")).toBeDisplayed();
   await expect(driver.$("#average-wait-time")).toBeDisplayed();
   ```

2. **Test Real-time Updates**

   ```javascript
   // Note initial queue length
   const initialQueueLength = await driver.$("#queue-length").getText();

   // Create new token (simulate customer action)
   // This would be done through API or separate test

   // Wait for dashboard update
   await driver.waitUntil(async () => {
     const newQueueLength = await driver.$("#queue-length").getText();
     return newQueueLength !== initialQueueLength;
   }, 10000);

   // Verify update
   const updatedQueueLength = await driver.$("#queue-length").getText();
   await expect(updatedQueueLength).not.toBe(initialQueueLength);
   ```

3. **Test Dashboard Filters**

   ```javascript
   // Apply time filter
   await driver.$("#time-filter").click();
   await driver.$("#filter-today").click();

   // Verify filtered data
   await expect(driver.$("#filtered-metrics")).toBeDisplayed();

   // Apply counter filter
   await driver.$("#counter-filter").click();
   await driver.$("#filter-counter-1").click();

   // Verify counter-specific data
   await expect(driver.$("#counter-specific-metrics")).toBeDisplayed();
   ```

**Expected Results**:

- Dashboard displays accurate metrics
- Real-time updates work correctly
- Filters apply properly
- Data refreshes automatically

## 🔒 Security Scenarios

### Scenario 7: Access Control Testing

**Objective**: Test security and access control mechanisms

**Test Steps**:

1. **Test Unauthorized Access**

   ```javascript
   // Try to access admin features without login
   await driver.url("http://localhost:3000/admin/users");

   // Verify redirect to login
   await expect(driver.$("#login-form")).toBeDisplayed();
   ```

2. **Test Role-based Access**

   ```javascript
   // Login as staff user
   await driver.$("#username").setValue("staff");
   await driver.$("#password").setValue("staff123");
   await driver.$("#submit-login").click();

   // Try to access admin features
   await driver.$("#admin-features").click();

   // Verify access denied
   await expect(driver.$("#access-denied-message")).toBeDisplayed();
   ```

3. **Test Session Management**

   ```javascript
   // Login as admin
   await driver.$("#username").setValue("admin");
   await driver.$("#password").setValue("admin123");
   await driver.$("#submit-login").click();

   // Wait for session timeout (simulate)
   await driver.pause(30000);

   // Try to perform admin action
   await driver.$("#create-user-button").click();

   // Verify session expired
   await expect(driver.$("#session-expired-message")).toBeDisplayed();
   ```

**Expected Results**:

- Unauthorized access is blocked
- Role-based permissions enforced
- Session management works correctly
- Security messages are clear

## ⚡ Performance Scenarios

### Scenario 8: Load Testing

**Objective**: Test system performance under load

**Test Steps**:

1. **Test Multiple Concurrent Users**

   ```javascript
   // Simulate multiple admin users
   const concurrentUsers = 5;
   const promises = [];

   for (let i = 0; i < concurrentUsers; i++) {
     promises.push(createAdminSession(`admin${i}`, `admin${i}123`));
   }

   // Wait for all sessions
   await Promise.all(promises);

   // Verify all users can access dashboard
   for (let i = 0; i < concurrentUsers; i++) {
     await expect(driver.$(`#admin-dashboard-${i}`)).toBeDisplayed();
   }
   ```

2. **Test Large Data Sets**

   ```javascript
   // Create large number of users
   for (let i = 0; i < 100; i++) {
     await driver.$("#create-user-button").click();
     await driver.$("#new-username").setValue(`user${i}`);
     await driver.$("#new-email").setValue(`user${i}@test.com`);
     await driver.$("#new-password").setValue("password123");
     await driver.$("#submit-user").click();
   }

   // Verify pagination works
   await expect(driver.$("#pagination-controls")).toBeDisplayed();

   // Test page navigation
   await driver.$("#next-page").click();
   await expect(driver.$("#page-2")).toBeDisplayed();
   ```

**Expected Results**:

- System handles concurrent users
- Large data sets display correctly
- Pagination works properly
- Performance remains acceptable

## 📱 Mobile-Specific Scenarios

### Scenario 9: Mobile Admin Interface

**Objective**: Test admin portal on mobile devices

**Test Steps**:

1. **Test Mobile Layout**

   ```javascript
   // Set mobile viewport
   await driver.setWindowSize(375, 667);

   // Verify mobile layout
   await expect(driver.$("#mobile-menu")).toBeDisplayed();
   await expect(driver.$("#desktop-menu")).not.toBeDisplayed();
   ```

2. **Test Touch Interactions**

   ```javascript
   // Test swipe navigation
   await driver.touchAction([
     { action: "press", x: 300, y: 300 },
     { action: "moveTo", x: 100, y: 300 },
     { action: "release" },
   ]);

   // Verify navigation worked
   await expect(driver.$("#swiped-content")).toBeDisplayed();
   ```

3. **Test Mobile Forms**

   ```javascript
   // Test mobile form input
   await driver.$("#mobile-username-input").click();
   await driver.$("#mobile-username-input").setValue("mobileadmin");

   // Verify mobile keyboard
   await expect(driver.$("#mobile-keyboard")).toBeDisplayed();
   ```

**Expected Results**:

- Mobile layout adapts correctly
- Touch interactions work properly
- Mobile forms function correctly
- Responsive design works

---

**Last Updated**: $(date)
**Version**: 1.0.0
**Maintainer**: Development Team
