The MemoTrack application's frontend is a **React single-page application (SPA)** designed as a Software-as-a-Service (SaaS) solution for hospitals [Documentation - React App too]. The React app is built using **Create React App**.

### Getting Started and Development

The React app is bootstrapped with Create React App, providing standard scripts for development and deployment.

*   **`npm start`**: Runs the application in development mode, typically accessible at `http://localhost:3000`. The page automatically reloads upon code changes.
*   **`npm run build`**: Compiles the application for production, generating a minified and optimised build in the `build` folder. This output is ready for deployment.

### Core Architecture and Technologies

The React frontend follows a "hospital-first" architecture and leverages modern web technologies for a responsive and secure experience.

*   **Hospital-First Architecture**:
    *   The application initiates with **hospital selection**.
    *   Information about the **selected hospital** is stored in `localStorage` and is crucial for subsequent login attempts and all API requests.
*   **Authentication and User Management**:
    *   Authentication is **JWT-based**, interacting with the Django REST Framework backend.
    *   Upon successful login, the `authToken`, user roles, and profile data are **stored in `localStorage`** for persistence.
    *   An Axios interceptor is configured to automatically **attach the authentication token** to all outgoing API requests.
    *   The `AuthContext` and `useAuth` custom hook manage the user, hospital, and token states, facilitating global access to authentication status and user roles (e.g., `isSuperuser`, `isApprover`, `isResponder`, `isCreator`, `isRaiser`).
*   **Role-Based Routing**:
    *   After authentication, the application routes users based on their assigned roles, using flags such as `is_super_user`, `is_approver`, `is_responder`, and `is_raiser`.
    *   The current implementation prioritises the **`superuser` dashboard**, with other role-specific pages serving as placeholders for future expansion.
*   **Styling and UI Components**:
    *   The application uses **Tailwind CSS** for utility-first styling and ensures **responsiveness** across desktop, tablet, and mobile screens.
    *   UI components are built using **Mantine**, including packages like `@mantine/core`, `@mantine/hooks`, and `@mantine/notifications`.
    *   Icons are provided by **Lucide React**.
*   **Page Structure and Routing**:
    *   Key pages include:
        *   **Login Page (`LoginPage.jsx`)**: Handles user authentication and allows users to set new passwords.
        *   **Dashboard Content (`DashboardContent.jsx`)**: Provides administrative interfaces for managing users, roles, blocks, approver hierarchies, and shifts.
        *   **Memos Page (`MemosPage.jsx`)**: Displays a list of memos.
        *   **Memo Detail Page (`MemoDetail.jsx`)**: Shows detailed information about a specific memo.
        *   **Ward Manager (`WardManager.jsx`)**: Manages wards associated with a particular block.
    *   Client-side routing is configured via the `.htaccess` file to redirect specific paths (e.g., `/users`, `/roles`, `/blocks`, `/shifts`, `/hierarchies`, `/memos`, `/memo/:memoId`, `/wards/:blockId`) to `index.html`.
*   **Code Structure**:
    *   The recommended project structure for the frontend includes `src/api/` for Axios client and API calls, `src/auth/` for login/logout logic, `src/context/` for Auth and hospital context, `src/pages/` for page components, `src/routes/` for role-based routing, and `src/utils/` for helper functions.
    *   The current file structure organises components, contexts, hooks, services, and utilities within the `src/` directory.
*   **API Client**:
    *   An `ApiService` class centralises all backend API interactions.
    *   It supports standard HTTP methods (GET, POST, PATCH, DELETE) and manages the `Authorization` header with the authentication token.
    *   The `baseURL` for API requests is configured via environment variables (`process.env.BACKEND_URL`) or defaults to `https://app.memotrack.net/`.
*   **State Management and Hooks**:
    *   Beyond `AuthContext`, the application uses various custom React Hooks:
        *   **`useCrud`**: Simplifies Create, Read, Update, and Delete operations for different entities by abstracting API calls.
        *   **`useForm`**: Manages form state, handles changes, blur events, and validates input fields based on defined schemas.
        *   **`useFlutter`**: Provides context for communication with the Flutter environment when the React app is embedded within a WebView.

### Integration with Flutter

The React frontend is designed to be seamlessly embedded and interact with a Flutter mobile application via a WebView.

*   **Data Injection**: When loaded in Flutter, the React app receives a `source=flutter` query parameter. The Flutter app injects user and authentication data (user details, hospital info, `authToken`, and `flutter_version`) into the React app's `localStorage` via a `memo_user_ready` event.
*   **Communication Channels**: A `ReactToFlutter` JavaScript channel enables the React app to send messages back to the Flutter host, supporting actions like `updateUser`, `showMessage`, and `uploadImage` (for memos).

### Important Considerations for Developers

*   **`localStorage` Check**: Always verify the presence of existing hospital and user data in `localStorage` before determining navigation routes.
*   **Development Focus**: Initial development efforts should concentrate on implementing **hospital selection**, the **login process**, and the **superuser dashboard**. Other role-specific features can be added later as "Coming Soon" functionalities.


Here is a comprehensive documentation of the files and directories within the `frontend/app/src/` directory of the React application:

### `src/` Directory Structure Overview

The `src/` directory is the core of the React application, organising its components, state management, API interactions, and utility functions into logical folders.

*   `src/App.css`
*   `src/App.js`
*   `src/App.test.js`
*   `src/index.css`
*   `src/index.js`
*   `src/output.css`
*   `src/reportWebVitals.js`
*   `src/setupTests.js`
*   `src/components/`
*   `src/contexts/`
*   `src/hooks/`
*   `src/services/`
*   `src/utils/`

---

### Individual File and Folder Documentation:

#### **Root Level Files in `src/`**

1.  **`App.css`**
    *   **Purpose**: Contains standard CSS styles for the main `App` component. It includes basic styling for the app's logo, header, and links.
    *   **Details**: Defines animations (e.g., `App-logo-spin`) and responsive styles using media queries.
    *   **Usage**: Imported by `App.js`.

2.  **`App.js`**
    *   **Purpose**: This is the **main entry point** for the React application's routing and a central orchestrator for core functionalities like authentication and Flutter integration.
    *   **Key Functionality**:
        *   **Routing**: Uses `react-router-dom` to define client-side routes, including `/` (dashboard), `/users`, `/roles`, `/blocks`, `/shifts`, `/hierarchies`, `/memos`, `/memo/:memoId`, and `/wards/:blockId`.
        *   **Authentication**: Utilises `AuthContext` and the `useAuth` hook to determine if a user is authenticated. It conditionally renders the `LoginPage` or the `DashboardLayout` based on the authentication status.
        *   **Flutter Integration**: Employs `useFlutterBridge` to detect if the app is embedded within a Flutter WebView (via `?source=flutter` query parameter) and handles the injection of user data from Flutter into `localStorage` upon a `memo_user_ready` event.
    *   **Dependencies**: Imports from `react-router-dom`, `react-hot-toast`, `AuthContext`, `FlutterContext`, `useAuth`, `LoginPage`, `DashboardLayout`, `WardManager`, `MemosPage`, `MemoDetail`.

3.  **`App.test.js`**
    *   **Purpose**: Contains boilerplate unit tests for the `App` component, typically generated by Create React App.
    *   **Details**: Uses `@testing-library/react` and `@testing-library/jest-dom` for testing React components.

4.  **`index.css`**
    *   **Purpose**: Imports the Tailwind CSS framework, indicating that the application uses a utility-first approach for styling.
    *   **Usage**: Imported by `index.js` to apply global styles.

5.  **`index.js`**
    *   **Purpose**: The primary JavaScript entry point for the entire React application. It renders the root React component (`MemoTrackApp`) into the HTML DOM.
    *   **Key Functionality**:
        *   **React DOM Rendering**: Uses `ReactDOM.createRoot` to render the application.
        *   **Mantine Provider**: Wraps the application with `MantineProvider` to enable Mantine UI components and hooks, which are used for building the user interface.
        *   **Web Vitals Reporting**: Integrates `reportWebVitals` for performance measurement.
    *   **Dependencies**: Imports `React`, `ReactDOM`, `index.css`, `MemoTrackApp`, `reportWebVitals`, and `MantineProvider`.

6.  **`output.css`**
    *   **Purpose**: This file is typically the **compiled output** of Tailwind CSS, containing all the generated utility classes and base styles used in the application.
    *   **Details**: It includes definitions for colours (e.g., `var(--color-red-50)`), spacing (`var(--spacing)`), font families, and various utility classes like `mt-6`, `flex`, `bg-red-50`, `text-center`, and responsive breakpoints (`sm:`, `md:`, `lg:`).

7.  **`reportWebVitals.js`**
    *   **Purpose**: A utility file for measuring and reporting core web vitals (CLS, FID, FCP, LCP, TTFB), crucial for performance monitoring.
    *   **Usage**: Integrated into `index.js`.

8.  **`setupTests.js`**
    *   **Purpose**: Configuration file for Jest and React Testing Library, often used to extend Jest's matchers for DOM assertions.
    *   **Details**: Imports `@testing-library/jest-dom`.

---

#### **`src/components/` Directory**

This directory houses reusable UI components, organised by their functional area.

1.  **`src/components/auth/`**
    *   **`LoginPage.jsx`**:
        *   **Purpose**: Handles user authentication and allows users to set new passwords.
        *   **Functionality**: Manages phone number, password, and confirm password states. It interacts with the `useAuth` hook to perform login and password setup, displaying loading states and error messages. It can toggle between login and set password forms.
    *   **`SetPassword.jsx`**:
        *   **Purpose**: Dedicated component for setting a new password.
        *   **Functionality**: Takes phone number, new password, and confirm password as input. It validates inputs (e.g., passwords matching, minimum length) and calls `apiService.setPassword` to update the password. It uses `useNavigate` for potential redirection after success.

2.  **`src/components/common/`**
    *   **`CrudTable.jsx`**:
        *   **Purpose**: A generic table component designed to display and manage data for various entities (e.g., users, roles, blocks) with Create, Read, Update, and Delete (CRUD) actions.
        *   **Functionality**: Renders data dynamically based on `columns` prop, provides "Add", "Edit", "Delete", and "View" buttons, and includes a specific "Manage Wards" link for "Blocks".
        *   **Dependencies**: Uses Lucide React icons (`Edit`, `Eye`, `Plus`, `Trash2`).
    *   **`LoadingSpinner.jsx`**:
        *   **Purpose**: A simple component to display a loading message and spinner while data is being fetched or processed.
        *   **Dependencies**: Uses `Building2` icon from Lucide React.
    *   **`WardSelectDialog.jsx`**:
        *   **Purpose**: A modal dialog for selecting a ward, often used in memo creation or user block/ward switching flows.
        *   **Functionality**: Fetches and displays wards grouped by block. It supports searching and filtering wards within active blocks. It allows a user to select a ward and confirms their selection before proceeding.

3.  **`src/components/dashboard/`**
    *   **`DashboardContent.jsx`**:
        *   **Purpose**: Renders the main content of the dashboard based on the `activeTab` selected (e.g., 'users', 'roles', 'blocks', 'shifts', 'hierarchies').
        *   **Functionality**: Dynamically renders `CrudTable` for different entities. It manages modal states for creating and editing items and confirmation dialogs for deletion. Access control is based on `isSuperuser` or `isStaff` roles.
        *   **Dependencies**: Uses `useCrud` hook for data management and various form components (`UserForm`, `RoleForm`, etc.).
    *   **`DashboardIframe.jsx`**:
        *   **Purpose**: Displays precomputed dashboard data fetched from the backend, embedded within an iframe.
        *   **Functionality**: Constructs the URL for the dashboard JSON data based on `hospitalId`, selected period (`today`, `week`, `month`), and selected date. It fetches the JSON data and displays loading/error states. It also includes navigation controls (previous/next period, refresh).
    *   **`DashboardLayout.jsx`**:
        *   **Purpose**: Provides the overall layout for the authenticated user's dashboard, including the header, sidebar navigation, and main content area.
        *   **Functionality**: Manages sidebar open/close state, displays user and hospital information, and handles logout. It uses `useLocation` to determine the active tab and passes it down to `Navigation` and `Outlet` (for nested routes). It also handles Flutter integration awareness by setting `localStorage` flags.
    *   **`Navigation.jsx`**:
        *   **Purpose**: Renders the sidebar navigation links for the dashboard.
        *   **Functionality**: Displays links for 'Dashboard', 'Memos', 'Users', 'Roles', 'Blocks', 'Hierarchies', 'Shifts', and 'Settings'. The visibility of 'Users', 'Roles', 'Blocks', 'Hierarchies', and 'Shifts' is conditional based on `isSuperuser` or `isStaff` roles. It highlights the active tab and provides a mechanism to close the mobile sidebar on navigation item click.

4.  **`src/components/forms/`**
    *   These components are specific forms for creating or editing various entities within the application. They typically use the `useForm` hook for input management and validation, and `FormField` for rendering individual input elements.
    *   **`ApproverHierarchyForm.jsx`**: Manages the creation and editing of approver hierarchies, including `is_active` status and multiple approval levels with roles and priorities.
    *   **`BlockForm.jsx`**: Form for creating or editing a block, including name, number of floors, and description.
    *   **`RoleForm.jsx`**: Form for creating or editing user roles, specifying name, description, and permissions (can approve, respond, create memos).
    *   **`ShiftForm.jsx`**: Form for creating or editing shifts, including name, start time, end time, and description.
    *   **`UserForm.jsx`**: Form for creating or editing user accounts, including institution ID, role, and phone number. It dynamically fetches available roles from the API for selection.

5.  **`src/components/memos/`**
    *   **`BackButton.jsx`**:
        *   **Purpose**: A reusable "Back" button component that navigates to the previous page in history or a fallback route if no history is available.
    *   **`MemoDetail.jsx`**:
        *   **Purpose**: Displays detailed information about a specific memo, including its latest snapshot, hierarchy, and worker status.
        *   **Functionality**: Fetches memo data using `memoId` from URL parameters. It conditionally displays "Edit" button for creators and integrates `MemoInfo`, `MemoTimeline`, and `WorkerStatus` components.
    *   **`MemoEditForm.jsx`**:
        *   **Purpose**: Form for updating an existing memo, allowing changes to complaint, department, and tagged role.
        *   **Functionality**: Fetches roles for the "Tagged Role" dropdown and sends a PATCH request to the API to update the memo's payload.
    *   **`MemoForm.jsx`**:
        *   **Purpose**: Form for creating a new memo.
        *   **Functionality**: Guides the user through confirming/changing their current ward, then allows input for complaint, department, and tagging a role. It generates a UUID for the memo and uses `apiService.createMemo` to submit. It also interacts with `useFlutter` to update user info in the Flutter app if embedded.
    *   **`MemoInfo.jsx`**:
        *   **Purpose**: Displays general information about a memo from its latest snapshot, grouped into "Request Info" and "User Info" sections.
        *   **Functionality**: Renders memo details, including complaint, department, tagged role, user role, phone number, block, ward, and floor. It also includes functionality for refreshing OTPs (Attendee and Completion OTPs) for relevant user roles.
    *   **`MemosPage.jsx`**:
        *   **Purpose**: Displays a list of memos, categorised by user role (creator, approver, responder, admin).
        *   **Functionality**: Fetches memos using `apiService.getUserMemos`. It provides a "Create" button for creators and navigates to `MemoDetail` on memo click. It shows a count of active memos and handles loading/error states. It uses `MemoTabbedList` for display and `MemoForm` for creation.
    *   **`MemoTimeline.jsx`**:
        *   **Purpose**: Displays the approval timeline/hierarchy for a memo and allows users to approve or reject steps.
        *   **Functionality**: Renders each step of the approval hierarchy, indicating its status (approved/pending). It provides "Approve" and "Undo Approval" buttons conditionally based on the current user's role (`currentUserRole`, `isCurrentUserSuperuser`) and whether they previously approved the step.
    *   **`PhoneLink.jsx`**:
        *   **Purpose**: Renders a clickable phone number that attempts to initiate a call.
        *   **Functionality**: Dynamically enables/disables the link based on whether the app is running in a Flutter WebView and the Flutter app's version (`flutter_version`) to ensure proper phone call handling within the mobile environment.
    *   **`WorkerStatus.jsx`**:
        *   **Purpose**: Displays and allows adding/deleting worker status updates (comments) for a memo, including speech-to-text functionality.
        *   **Functionality**: Processes raw status data to display it with threading (replies). It provides a modal for adding new updates, which can include comments, status type (`attended`, `completed`, `incomplete`, `tagged`), tagged roles, and OTP validation for specific status types (e.g., `completed`, first-time comments). It integrates with the Flutter app for speech-to-text capabilities, sending speech commands and receiving results.

6.  **`src/components/modals/`**
    *   **`ConfirmDialog.jsx`**:
        *   **Purpose**: A generic modal component for displaying confirmation messages before executing a potentially destructive action (e.g., deletion).
        *   **Functionality**: Takes title, message, confirm/cancel text, and loading state as props.

7.  **`src/components/prototype/`**
    *   **`MemoExplorer.jsx`**:
        *   **Purpose**: An experimental UI component for exploring memo lists with client-side search and filtering.
        *   **Functionality**: Organises memos by roles and tabs, offering dynamic filtering options for wards and departments, and a search bar for complaints. Built with Mantine components.
    *   **`MemoExplorerTW.jsx`**:
        *   **Purpose**: A Tailwind CSS-only version of `MemoExplorer` for exploring memo lists, offering the same filtering capabilities without Mantine dependencies.
        *   **Functionality**: Provides role and tab selectors, search, and filters for ward and department.
    *   **`MemoTabbedList.jsx`**:
        *   **Purpose**: Displays memos in a tabbed list format, allowing users to filter and search for memos within different categories.
        *   **Functionality**: Organises memos into tabs (e.g., "All"), provides advanced filtering options by complaint, ward, block, floor, department, and date, and supports searching.

8.  **`src/components/ui/`**
    *   **`Checkbox.jsx`**: A reusable checkbox input component with a label.
    *   **`FormField.jsx`**: A versatile component for rendering various input types (text, select, textarea, time) with integrated labels, error display, and required indicators.
    *   **`Modal.jsx`**: A generic modal component for displaying content in a pop-up window, supporting different sizes.

9.  **`src/components/wards/`**
    *   **`WardManager.jsx`**:
        *   **Purpose**: Manages wards associated with a particular block.
        *   **Functionality**: Fetches and displays wards for a given `blockId`. Allows adding new wards (with name and floor) and deleting existing ones. Wards are grouped and sortable by floor, with an expand/collapse feature.

---

#### **`src/contexts/` Directory**

Contains React Contexts for global state management.

1.  **`AuthContext.js`**
    *   **Purpose**: Provides global access to authentication-related data and functions throughout the application.
    *   **Functionality**: Manages `user`, `hospital`, `authToken`, and `loading` states. It offers methods like `login`, `logout`, `updateAuth`, and `setPasswordfunc`. It also exposes derived states such as `isAuthenticated`, `isSuperuser`, `isApprover`, `isResponder`, `isCreator`, and `isRaiser`.
    *   **Usage**: The `useAuth` hook is used to consume this context.
2.  **`FlutterContext.jsx`**
    *   **Purpose**: Provides a mechanism for the React app to **communicate with its Flutter host** when embedded within a WebView.
    *   **Functionality**: Offers methods like `sendToFlutter`, `updateUserInFlutter`, and `showFlutterMessage`, which send messages to the Flutter `ReactToFlutter` JavaScript channel.

---

#### **`src/hooks/` Directory**

Contains custom React hooks for reusable logic.

1.  **`useAuth.js`**
    *   **Purpose**: A custom hook that simplifies access to the `AuthContext` data and functions.
    *   **Usage**: Ensures that the `useAuth` hook is called within an `AuthProvider`.
2.  **`useCrud.jsx`**
    *   **Purpose**: A generic custom hook for abstracting common Create, Read, Update, and Delete (CRUD) operations for different entities.
    *   **Functionality**: Takes an `entityType` (e.g., 'users', 'roles') and provides `data`, `loading`, `error`, `fetchData`, `createItem`, `updateItem`, and `deleteItem` functions, leveraging `apiService` from `useAuth`.
3.  **`useForm.jsx`**
    *   **Purpose**: A custom hook for managing form state, input changes, blur events, and validation based on a provided schema.
    *   **Functionality**: Provides `values`, `errors`, `touched`, `handleChange`, `handleBlur`, `validate`, `reset`, and `setValues` for streamlined form handling. It includes specific casting for role selection to integer.

---

#### **`src/services/` Directory**

Contains services for interacting with external APIs.

1.  **`api.js` (ApiService)**
    *   **Purpose**: A centralized class that handles all interactions with the Django REST Framework backend API.
    *   **Key Functionality**:
        *   **Base URL Management**: Configures the `baseURL` for API requests, defaulting to `https://app.memotrack.net/` or using `process.env.BACKEND_URL`.
        *   **Request Handling**: The `request` method supports GET, POST, PATCH, and DELETE HTTP methods, and automatically attaches the `Authorization` header with the JWT token retrieved from `localStorage`.
        *   **Specific Endpoints**: Provides methods for various API interactions, including `login`, `logout`, `setPassword`, `getHospital`, `getUsers`, `createUser`, `updateUser`, `deleteUser`, and similar operations for roles, blocks, shifts, approver hierarchies, memos, worker statuses, and wards. It also includes `refreshOTP`.

---

#### **`src/utils/` Directory**

Contains general utility functions and constants.

1.  **`constants.js`**
    *   **Purpose**: Stores constants such as API endpoints and local storage keys for easy management and consistency across the application.
    *   **Details**: Defines `API_ENDPOINTS` (e.g., `HOSPITALS`, `TOKEN_AUTH`, `USERS`), `USER_ROLES` (e.g., `SUPERUSER`, `APPROVER`, `RESPONDER`, `RAISER`), and `LOCAL_STORAGE_KEYS` (`authToken`, `user`, `selectedHospital`).
