# Build in Five — Add Frontend Pages

## Purpose
Use this prompt AFTER generating the skeleton app to add domain-specific frontend pages. Ensures consistent UI patterns, Tailwind styling, and auth integration across all Build in Five applications.

## How to use
1. Skeleton app generated via `01-skeleton-app-scaffold.md`
2. Backend API endpoints created via `02-add-backend-api.md` (so the frontend has endpoints to call)
3. Fill in placeholders and paste into Cursor / GSD

---

## 1. Frontend Conventions (mandatory)

| Convention | Standard |
|-----------|----------|
| Components | Functional components only, no class components |
| State | useState, useReducer, useContext. No Redux. |
| Styling | Tailwind CSS utility classes only. No CSS modules, no styled-components. |
| API calls | Via `api/client.js` Axios instance (auto-injects auth token) |
| Loading states | Always show LoadingSpinner during async operations |
| Error handling | ErrorBoundary for crashes; inline error messages for API failures |
| Forms | Controlled components with validation before submit |
| Tables | Responsive with horizontal scroll on mobile; sortable headers; pagination controls |
| Colour palette | Moody's brand: navy `#1a2942`, teal `#00a4a6`, light grey `#f0f4f8`, white `#ffffff` |
| Typography | Font: system font stack (Tailwind default). Headings: font-bold. Body: text-sm/text-base. |
| Auth | All pages wrapped in ProtectedRoute. Role-gated UI via `useAuth()` hook. |
| Routing | React Router v6. Lazy loading for page components. |

---

## 2. The Prompt

```
I have an existing React 18 + Vite + Tailwind CSS application with Entra ID authentication (MSAL React) and an Axios API client with auth interceptor. I need to add frontend pages.

## PAGE REQUIREMENTS

Page name: [PAGE_NAME]
Route: /[route-path]
Description: [WHAT_THIS_PAGE_DOES]
Required role(s): [ROLES_THAT_CAN_ACCESS — e.g., "Admin, Editor" or "any authenticated"]
API endpoints used:
- GET /api/v1/[entities] (list with pagination)
- GET /api/v1/[entities]/{id} (detail)
- POST /api/v1/[entities] (create)
- PUT /api/v1/[entities]/{id} (update)
- DELETE /api/v1/[entities]/{id} (delete)

## GENERATE THE FOLLOWING

### frontend/src/pages/[PageName].jsx
Main page component with:
- Data fetching on mount using the useApi custom hook
- Loading state (LoadingSpinner) while fetching
- Error state with retry button
- Empty state with helpful message when no data exists

### frontend/src/components/[entity]/[Entity]Table.jsx
Data table component with:
- Column headers matching entity fields: [LIST_COLUMNS]
- Sortable columns (click header to toggle asc/desc)
- Pagination controls (Previous/Next, showing "1-20 of 150")
- Row click navigates to detail view (or opens modal)
- Action column with Edit and Delete buttons (visible based on user role)
- Responsive: horizontal scroll wrapper on small screens
- Tailwind styling:
  - Header row: bg-gray-50 text-xs font-medium text-gray-500 uppercase tracking-wider
  - Data rows: hover:bg-gray-50, alternating bg-white/bg-gray-25
  - Borders: divide-y divide-gray-200

### frontend/src/components/[entity]/[Entity]Form.jsx
Create/Edit form component with:
- Controlled inputs for each editable field
- Field validation (required fields, type checks, max lengths)
- Submit button with loading state
- Cancel button that navigates back
- Used for both Create (empty form) and Edit (pre-populated) via props
- Tailwind form styling:
  - Labels: block text-sm font-medium text-gray-700
  - Inputs: mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-teal-500 focus:ring-teal-500 sm:text-sm
  - Error messages: mt-1 text-sm text-red-600
  - Submit button: bg-[#00a4a6] hover:bg-[#008a8c] text-white font-medium py-2 px-4 rounded-md

### frontend/src/components/[entity]/[Entity]Detail.jsx (if needed)
Detail view component with:
- Full entity display in a card layout
- Edit and Delete action buttons (role-gated)
- Back navigation
- Metadata footer: created by, created at, last updated

### frontend/src/pages/[PageName].jsx integration
- Wrap route in ProtectedRoute with required role(s)
- Add to App.jsx router configuration
- Add to Sidebar navigation

### frontend/src/hooks/use[Entity].js (optional)
Custom hook encapsulating:
- List fetch with pagination, filter, and sort state
- Single entity fetch
- Create, update, delete mutations
- Loading and error state per operation

## UI PATTERNS

### Filter bar (if applicable)
- Horizontal bar above the table
- Dropdown filters for: [FILTER_FIELDS]
- Search input for text search
- "Clear filters" button
- Filters applied on change (no separate "Apply" button)
- Tailwind: flex items-center gap-4 p-4 bg-white border-b border-gray-200

### Action confirmation
- Delete: show confirmation modal "Are you sure you want to delete [entity name]?"
- Modal: fixed overlay with centered white card, "Cancel" and "Delete" buttons
- Delete button: bg-red-600 hover:bg-red-700 text-white

### Toast notifications
- Success: green banner, auto-dismiss after 3 seconds
- Error: red banner, manual dismiss
- Position: top-right, stacked
- Tailwind: fixed top-4 right-4 z-50

### Page layout
```jsx
<div className="px-6 py-4">
  {/* Page header */}
  <div className="flex items-center justify-between mb-6">
    <h1 className="text-2xl font-bold text-[#1a2942]">[Page Title]</h1>
    <button className="bg-[#00a4a6] hover:bg-[#008a8c] text-white font-medium py-2 px-4 rounded-md">
      + Create New
    </button>
  </div>

  {/* Filter bar */}
  {/* ... */}

  {/* Data table */}
  {/* ... */}
</div>
```

## CRITICAL CONSTRAINTS
- Tailwind CSS only — no inline styles, no CSS files, no styled-components
- All API calls via the Axios client in api/client.js (never raw fetch)
- All pages must handle: loading, error, empty, and data states
- All destructive actions (delete) require confirmation
- Role-gated UI: hide buttons/actions the user's role cannot perform (don't just disable them)
- No console.log statements in production code
- All text content in JSX (no i18n library needed, but don't hardcode text in multiple places)
- Colour palette: use Moody's brand colours defined above, extended via tailwind.config.js
```

---

## 3. Example: Exposure Records List Page

```
Page name: ExposureRecords
Route: /exposure-records
Description: Paginated list of ingested exposure records with filtering by geography, peril, and validation status. Click row to view detail. Create button opens upload form.
Required role(s): any authenticated
API endpoints used:
- GET /api/v1/exposure-records (list with pagination)
- GET /api/v1/exposure-records/{id} (detail)
- DELETE /api/v1/exposure-records/{id} (delete — Admin only)

Columns: source_file, client_name, geography, peril, line_of_business, tiv (formatted as currency), validation_status (badge: green/amber/red), created_at
Filter fields: geography (dropdown), peril (dropdown), validation_status (dropdown)
```
