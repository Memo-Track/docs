The MemoTrack Flutter application is designed as a **comprehensive hospital management application**, aimed at efficiently managing and tracking memos within a hospital setting.

### App Specification and Requirements

*   **Application Name**: The project is internally named `hospital_management` in `pubspec.yaml`, but the Android application label is **`MemoTrack`**, and its package name is `com.memotrack.myapp`.
*   **Version**: The application is at **version 2.0.1+11**.
*   **Environment**: It is built for **Flutter SDK versions '>=3.0.0 <4.0.0'**.
*   **Core Functionality**:
    *   User authentication with role-based access.
    *   Management of hospital infrastructure elements like blocks and wards.
    *   Display and interaction with memos, potentially via an embedded web (React) application.
    *   Notifications via Firebase Cloud Messaging (FCM).
    *   Speech-to-text functionality for input.
    *   Image upload associated with memos.
*   **Firebase Integration**:
    *   Uses **Firebase Core**, **Firebase Messaging**, and **Firebase Remote Config**.
    *   The project ID is **`memo-track`**.
    *   Firebase Messaging is configured to handle foreground, background, and terminated state notifications using `flutter_local_notifications`.
    *   Remote Config is used to fetch dynamic configurations, including the backend and frontend (React) API URLs.
*   **API Communication**:
    *   Communicates with a backend API, with a **base URL primarily configured via Firebase Remote Config** (defaulting to `https://memotrack.pythonanywhere.com`).
    *   Supports various API endpoints for authentication, user management, hospital data, roles, blocks, shifts, wards, and memo operations (create, update, approve, reject, worker status).
*   **Frontend Integration (React)**:
    *   The app embeds a **React web application using `webview_flutter`** to display complex UI, especially for memo lists and details.
    *   A JavaScript channel (`ReactToFlutter`) enables two-way communication between the Flutter app and the embedded React app for actions like updating user data or triggering image uploads.
    *   Another JavaScript channel (`SpeechToText`) is used to send speech recognition results and status to the React app.
    *   Authentication tokens and user data are injected into the WebView's `localStorage` for the React app to utilise.
*   **Platform Specific Features**:
    *   Handles `tel:`, `sms:`, and `mailto:` URLs natively.
    *   Retrieves SIM card phone numbers using platform channels (`sim_info_channel`).
*   **Assets**: Includes various images (`ic_launcher.png`, `app_logo.png`) and a Lottie animation (`no_internet.json`).
*   **Build Automation**: Utilises GitHub Actions for continuous integration, including building Android App Bundles (AAB) and APKs, and for manual production builds with configurable environment and API URLs.

### Dependencies from `pubspec.yaml`

*   **State Management**: `provider`
*   **HTTP Requests**: `http`
*   **Local Storage**: `shared_preferences`
*   **UI Components**: `cupertino_icons`, `webview_flutter`, `animated_text_kit`, `lottie`
*   **Notifications**: `firebase_messaging`, `firebase_core`, `flutter_local_notifications`
*   **Permissions**: `permission_handler`
*   **Voice Input**: `speech_to_text`, `translator`
*   **Remote Configuration**: `firebase_remote_config`
*   **Platform Integrations**: `android_intent_plus`, `connectivity_plus`, `image_picker`, `path`, `url_launcher`, `package_info_plus`

### Dev Dependencies from `pubspec.yaml`

*   `flutter_test`
*   `flutter_lints`
*   `flutter_launcher_icons`

---

### `lib` Directory File Documentation

The `lib` directory is the heart of the Flutter application, containing all the Dart source code.

*   **`lib/main.dart`**
    *   **Purpose**: This is the **application's entry point**. It handles the initial setup including Firebase, Firebase Messaging for push notifications, remote configuration, and ensures essential notification permissions are requested. It then launches the main `MyApp` widget.
    *   **Key Contents**:
        *   `main()`: Asynchronously initialises `WidgetsFlutterBinding`, Firebase, and sets up Firebase Messaging specifically for Android and iOS platforms.
        *   `setupFirebaseMessaging()`: Configures **FCM for the app**, handling token retrieval, token refresh listeners, and setting up `flutter_local_notifications` to display foreground notifications and handle notification taps (routing to `MemoWebViewPage`).
        *   `_firebaseMessagingBackgroundHandler()`: A crucial top-level function that ensures Firebase is initialised to process messages when the app is in the background.
        *   `_handleMessage()`: Navigates the app to the `MemoWebViewPage` to display a specific memo when a notification containing a `memo_id` is tapped.
        *   `MyApp`: The root widget, responsible for setting up the `AuthProvider` for global state management, defining the application's material design theme (`AppTheme.lightTheme`), and designating `WelcomePage` as the initial screen.

*   **`lib/welcome.dart`**
    *   **Purpose**: Serves as the **initial splash screen**, presenting animated text while performing critical startup checks such as internet connectivity and necessary device permissions before navigating to the login or home screen.
    *   **Key Contents**:
        *   `_WelcomePageState`: Manages **animations** (fade and slide) for the intro sequence.
        *   `_handleStartupSequence()`: Orchestrates the asynchronous execution of internet connectivity checks, permission requests, and the authentication flow.
        *   `_checkInternet()`: Verifies network availability using `connectivity_plus` and displays a **Lottie animation (`no_internet.json`)** if offline, providing options to retry or open network settings.
        *   `_checkPermissions()`: Requests crucial permissions like **microphone, storage, and phone** needed for app functionality.
        *   `_handleAuthAndRedirect()`: Utilises the `AuthProvider` to determine if a user is authenticated and directs them to `HomeWrapper` (if logged in) or `LoginScreen` (if not).

*   **`lib/components/switch_dialogs.dart`**
    *   **Purpose**: Provides **reusable Flutter dialogs for block and ward selection**. These dialogs embed a `WebViewWidget` to render interactive HTML content fetched from the API, enabling a dynamic selection experience.
    *   **Key Contents**:
        *   `BlockSelectionDialog`/`_BlockSelectionDialogState`: Displays a `WebViewWidget` to present a list of blocks. It fetches block data using `apiService.getBlocks` and communicates user selection back to Flutter via a **JavaScript channel (`FlutterChannel`)**.
        *   `BlockSwitchingDialog`: A wrapper around `BlockSelectionDialog` that includes logic to call the `apiService.switchBlock` endpoint and display snackbar confirmations/errors upon successful or failed block switching.
        *   `WardSelectionDialog`/`_WardSelectionDialogState`: Functions identically to `BlockSelectionDialog` but is tailored for ward selection, fetching data using `apiService.getWards`.
        *   `WardSwitchingDialog`: A wrapper for `WardSelectionDialog` that integrates the `apiService.switchWard` functionality.

*   **`lib/core/theme/app_theme.dart`**
    *   **Purpose**: Centralises the **visual styling and theming** for the entire application, ensuring a consistent look and feel across all screens.
    *   **Key Contents**:
        *   `AppFontSizes`: Defines a set of static font size constants.
        *   `AppTheme`: Provides the `lightTheme` configuration, which leverages `Material3` and customises `ColorScheme`, `scaffoldBackgroundColor`, `appBarTheme`, `cardTheme`, `elevatedButtonTheme`, `outlinedButtonTheme`, `textButtonTheme`, `inputDecorationTheme`, `drawerTheme`, `listTileTheme`, and `textTheme` using a defined palette of colours and font sizes.
        *   `AppSpacing` and `AppRadius`: Static constants for consistent padding, margins, and border radii.

*   **`lib/models/auth_response.dart`**
    *   **Purpose**: Defines Dart classes for **parsing authentication responses** from the API and for representing block data.
    *   **Key Contents**:
        *   `AuthResponse`: A model class holding the `token`, `User`, and `Hospital` objects received post-login, with `fromJson` and `toJson` methods for serialization.
        *   `Block`: A simple data model for a block, containing its `id`, `name`, `description`, and `hospitalId`.

*   **`lib/models/hospital.dart`**
    *   **Purpose**: Defines the **data structure for a Hospital object**, including its geographical and contact information.
    *   **Key Contents**:
        *   `Hospital`: Model class with fields for `id`, `name`, `geolocationPoint`, `radius`, and `address`, complete with `fromJson` and `toJson` methods.

*   **`lib/models/hospital_models.dart`**
    *   **Purpose**: Provides **specific data models for hospital infrastructure elements**, such as blocks and wards, distinct from the generic `Block` model in `auth_response.dart`.
    *   **Key Contents**:
        *   `HospitalBlock`: Models a block with `id`, `name`, `noOfFloors`, and `hospitalId`. Includes robust `fromJson` parsing.
        *   `HospitalWard`: Models a ward with `id`, `name`, `block` (ID), `floor`, and `hospitalId`, also with `fromJson` parsing.

*   **`lib/models/toast_model.dart`**
    *   **Purpose**: Offers a utility to **display custom, top-positioned toast messages** (e.g., for success or error notifications) without relying on external packages.
    *   **Key Contents**:
        *   `showTopToast()`: A function that creates a temporary `OverlayEntry` to display a custom `SnackBar`-like message, fading out after a set duration.

*   **`lib/models/user.dart`**
    *   **Purpose**: Defines the **comprehensive data model for a user** within the application, including their personal details, roles, and current operational context.
    *   **Key Contents**:
        *   `User`: A model with extensive fields such as `id`, `institutionId`, `role`, `phoneNumber`, FCM tokens, current block/ward/floor, hospital ID, and **role-based boolean flags** like `isApprover`, `isSuperuser`, `isResponder`, `isCreator`.
        *   `fromJson()`: Factory constructor for instantiating a `User` object from a JSON map.
        *   `copyWith()`: Provides a convenient way to create a new `User` instance with some fields updated while others remain the same.

*   **`lib/providers/auth_provider.dart`**
    *   **Purpose**: Manages the **global authentication state** of the user using the Provider package, ensuring user and hospital data persist across sessions via `shared_preferences`.
    *   **Key Contents**:
        *   Private fields: `_user`, `_hospital`, `_token`, `_isLoading`, `_error` store the current authentication state.
        *   Getters: Provide access to boolean flags reflecting the user's roles (`isSuperuser`, `isApprover`, `isResponder`, `isCreator`).
        *   `initializeAuth()`: Loads previously saved authentication data from `SharedPreferences` when the app starts.
        *   `login()`, `setPassword()`, `logout()`: Methods to handle authentication operations by interacting with `ApiService` and managing the local authentication state.
        *   `updateFCMToken()`, `updateCurrentBlock()`, `updateCurrentWard()`: Update user-specific data both locally and on the backend.
        *   `_saveAuthData()`, `_saveUserData()`: Helper methods to persist authentication and user data to `SharedPreferences`.
        *   `parseApiError()`: A utility to extract and format readable error messages from API responses.

*   **`lib/screens/block_selection_screen.dart`**
    *   **Purpose**: A screen that allows a user (specifically approvers, based on the `HomeWrapper` logic) to **select their current working block**, which then updates their profile and redirects them.
    *   **Key Contents**:
        *   `_BlockSelectionScreenState`: Manages the selected block's ID and name, lists available blocks, and handles loading/error states.
        *   `_loadBlocks()`: Fetches the list of blocks from the `ApiService` for the user's hospital.
        *   `_submit()`: Calls `ApiService().updateUser` and `ApiService().switchBlock` to update the user's block on the backend, then updates the `AuthProvider` and navigates the user accordingly (either back to the dashboard or to the `HomeWrapper`).

*   **`lib/screens/main_navigation.dart`**
    *   **Purpose**: Implements the **main bottom navigation** structure of the app, allowing users to switch between different sections (Dashboard, Memos). It also ensures that deep links from notifications navigate to the correct memo page.
    *   **Key Contents**:
        *   `_navigatorKeys`: A list of `GlobalKey`s, each associated with a separate `Navigator` for independent navigation stacks within each tab.
        *   `_selectedIndex`: Tracks the currently active tab.
        *   `_redirectToMemo()`: Specifically handles navigation to a `MemoWebViewPage` with a given `memoId`, typically invoked when a push notification is tapped.
        *   `_onItemTapped()`: Manages tab switching and popping to the root of a tab's navigation stack if the same tab is re-tapped.
        *   `PopScope`: Controls the app's back button behaviour, popping routes within the current tab or switching to the first tab if no more routes are available.

*   **`lib/screens/memo_list_page.dart`**
    *   **Purpose**: Displays a **searchable, filterable, and sortable list of memos**, providing a user-friendly interface for managing maintenance requests.
    *   **Key Contents**:
        *   State variables: `_allMemos`, `_filteredMemos`, `_searchQuery`, `_selectedDepartment`, `_sortBy`, `_sortAscending`, `_currentPage`, `_departments` manage memo data, filters, and pagination.
        *   `_loadMemos()`: Fetches memos from the `ApiService` and populates the lists, also extracting unique departments for filtering.
        *   `_applyFiltersAndSearch()`: Applies the active search query, department filter, and sorting order to the memo list.
        *   `_paginatedMemos`: A getter that provides the subset of memos for the current page.
        *   `_showFilterBottomSheet()`: Presents a modal bottom sheet (`_FilterBottomSheet`) for users to interact with filtering and sorting options.
        *   UI: Features an `AppBar` with filter and refresh actions, a persistent search bar, filter summary chips, memo count, and paginated list of memo cards.

*   **`lib/screens/memo_web_page.dart`**
    *   **Purpose**: **Embeds the React-based memo management interface** as a `WebViewWidget` within the Flutter application. It serves as the primary bridge for deep interaction between the native Flutter environment and the web frontend.
    *   **Key Contents**:
        *   `WebViewController _controller`: The core component for managing the WebView, enabling JavaScript execution, handling navigation requests (like `tel:`, `sms:`, `mailto:` URLs are opened natively), and managing communication channels.
        *   `reactUrl`: The URL of the React frontend application, obtained from `RemoteConfigService`.
        *   **Speech-to-text integration**: Includes setup for `SpeechToText` (requesting microphone permission, initialising `SpeechToText`, managing locales like Tamil and English), and methods (`_startListening`, `_stopListening`, `_changeLanguage`) to send **real-time speech results and status back to the React app** via the `SpeechToText` JavaScript channel.
        *   `_injectUserData()`: A crucial method that injects the Flutter app's **authentication token, user data, hospital data, and Flutter app version into the WebView's `localStorage`**, making it accessible to the React frontend.
        *   `_handleReactMessage()`: Listens for messages sent from the React app via the `ReactToFlutter` JavaScript channel. It handles actions like `updateUser` (to update the native Flutter user state) and `uploadImage` (to navigate to the native image upload screen).

*   **`lib/screens/splash_screen.dart`**
    *   **Purpose**: This file is **commented out** in the provided source code, indicating it's likely an **older or alternative splash screen implementation**. Its original intent was to handle initial app loading, authentication checks, and navigation. This functionality appears to have been superseded by `lib/welcome.dart`.

*   **`lib/screens/upload_memo_image.dart`**
    *   **Purpose**: Provides a dedicated screen for **users to capture or select an image and upload it**, associating it with a specific memo.
    *   **Key Contents**:
        *   `memoId`: The identifier for the memo to which the image will be linked.
        *   `_pickImage()`: Uses the `image_picker` package to allow users to choose an image from their camera or photo gallery.
        *   `_uploadImage()`: Handles the actual image upload using `http.MultipartRequest`, sending the image along with `memo_id`, `user_id`, and `user_name` as headers to a specified API endpoint (`https://memo-track.free.beeceptor.com/api/img`).

*   **`lib/screens/ward_selection_screen.dart`**
    *   **Purpose**: Enables users to **select their current working ward**, with wards organized by block and floor for easier navigation and selection. It supports searching and filtering.
    *   **Key Contents**:
        *   Data structures: `allWards`, `wardsByBlock`, `blockNames` manage the ward data and its organization.
        *   `_loadWards()`: Fetches ward data from the `ApiService`.
        *   `_organizeWardsByBlock()`: Groups wards by their associated block and sorts them for display.
        *   `_filterWards()`: Updates the displayed list of wards based on a user's search query.
        *   `_submit()`: Updates the user's selected ward on the backend via `ApiService` and updates the `AuthProvider`'s state.
        *   UI: Features a search bar, expandable `ExpansionTile` widgets for each block, and `ListTile`s for individual wards, allowing selection via radio buttons.

*   **`lib/screens/auth/login_screen.dart`**
    *   **Purpose**: Provides the **user login interface and logic**, including phone number and password input, form validation, and navigation after successful authentication.
    *   **Key Contents**:
        *   `_formKey`: Used for form validation.
        *   `_phoneController`, `_passwordController`: Text controllers for the input fields.
        *   `_checkConnectivityAndPermissions()`: Ensures the device has internet access and necessary permissions before attempting login.
        *   `_login()`: Calls `authProvider.login` with user credentials. Upon success, it navigates to `BlockSelectionScreen` (for approvers) or `HomeWrapper` (for others). On failure, it uses `showTopToast` to display an error message.
        *   `_navigateToSetPassword()`: Directs the user to the `SetPasswordScreen`.
        *   UI: Displays the app logo, title, a login form with phone number and password fields (including a password visibility toggle), and action buttons for login and setting a new password.

*   **`lib/screens/auth/set_password_screen.dart`**
    *   **Purpose**: Enables users to **set or reset their account password** using their phone number, with an integrated feature to pick phone numbers directly from the device's SIM cards.
    *   **Key Contents**:
        *   `_formKey`: For form validation.
        *   `_phoneController`, `_passwordController`, `_confirmPasswordController`: Controllers for input fields.
        *   `_setPassword()`: Calls `authProvider.setPassword` to update the password on the backend. Navigates to `HomeWrapper` on success or displays an error toast.
        *   `_selectSimNumber()`: Requests phone permissions and then uses `SimService.getSimNumbers()` to fetch and display available SIM card numbers for selection in a bottom sheet.
        *   `_removeCountryCode()`: A helper function to clean up phone numbers obtained from SIM cards by removing country codes.
        *   UI: Contains input fields for phone number (with SIM selection), new password, and confirm password (both with visibility toggles), along with buttons to set the password.

*   **`lib/screens/home/admin_home_screen.dart`**
    *   **Purpose**: (Partially implemented) Intended as an **administrative home screen** featuring a dashboard and a navigation drawer for various administrative sections.
    *   **Key Contents**:
        *   `_selectedIndex`, `_screens`, `_navigationItems`: Manage which screen is currently displayed and define the drawer navigation options.
        *   `_buildDrawer()`: Constructs the app's navigation drawer with a header and list tiles for different sections (e.g., Dashboard, Users, Profile). The "Users" and "Profile" sections are commented out but indicate planned functionality.

*   **`lib/screens/home/block_selection_screen.dart`**
    *   **Purpose**: This file is **commented out** and represents an **alternative or deprecated implementation** for block selection within the home modules. The active implementation for block selection is located at `lib/screens/block_selection_screen.dart`.

*   **`lib/screens/home/dashboard_screen.dart`**
    *   **Purpose**: Displays the **primary dashboard** for authenticated users, presenting a summary of user information, hospital details, current location (block/ward), and assigned permissions. It also provides options for relevant actions like switching locations or logging out.
    *   **Key Contents**:
        *   `_showBlockSwitchingForm()`: Directs to the `BlockSelectionScreen` (the active one) for users to change their current block.
        *   `_showWardSwitchingForm()`: Directs to the `WardSelectionScreen` for users to change their current ward.
        *   `_buildPermissionChip()`: A helper function to visually represent user permissions (Superuser, Approver, Responder, Creator) as chips.
        *   `_showLogoutDialog()`: Presents a confirmation dialog for user logout.
        *   UI: Organised into cards, it displays a welcome message with user ID and role, hospital name and address, current block and ward (if set), and a dynamic section for block/ward switching buttons based on the user's roles. A "Permissions" section clearly lists all assigned roles.

*   **`lib/screens/home/home_wrapper.dart`**
    *   **Purpose**: Acts as an **intermediary screen after authentication** to direct the user to the appropriate starting point in the application based on their role and current state.
    *   **Key Contents**:
        *   Conditional Navigation:
            *   If no user is authenticated, it renders `LoginScreen`.
            *   If the user is an approver (`auth.isApprover`) and has no `currentBlockId` set, it directs them to the `BlockSelectionScreen` to select a block.
            *   Otherwise (for all other authenticated users or approvers with a block set), it displays the `MainNavigation` widget, which contains the main app tabs.

*   **`lib/services/api_service.dart`**
    *   **Purpose**: Provides a **centralized interface for all interactions with the backend API**, handling HTTP requests, authentication token management, and response parsing. It uses a singleton pattern to ensure a single instance throughout the app.
    *   **Key Contents**:
        *   `baseURL`: A static variable that holds the API's base URL, dynamically set from `RemoteConfigService` during initialisation.
        *   Singleton Pattern: `ApiService._internal()` and `factory ApiService()` ensure only one instance is created.
        *   `_getToken()`, `_getHeaders()`: Private methods for retrieving the authentication token from `SharedPreferences` and constructing HTTP request headers.
        *   `_makeRequest()`, `_makeListRequest()`: Generic private methods that abstract the HTTP request logic (GET, POST, PATCH, DELETE) for single object and list responses, respectively. They handle status codes and JSON decoding, throwing exceptions for errors.
        *   **Public API Methods**: A comprehensive set of methods covering all API interactions, including:
            *   Authentication: `login()`, `setPassword()`, `logout()`.
            *   User/Hospital Data: `getHospital()`, `updateFCMToken()`, `getUser()`, `getUsers()`, `createUser()`, `updateUser()`, `deleteUser()`.
            *   Role Management: `getRoles()`, `createRole()`, `updateRole()`, `deleteRole()`.
            *   Infrastructure Management: `getBlocks()`, `createBlock()`, `updateBlock()`, `deleteBlock()`, `getShifts()`, `createShift()`, `updateShift()`, `deleteShift()`, `getWards()`.
            *   Location Switching: `switchBlock()`, `switchWard()`.
            *   Memo Operations: `getMemos()` (list), `createMemo()`, `updateMemo()`, `approveMemo()`, `rejectMemo()`, `addWorkerStatus()`, `deleteWorkerStatus()`, `refreshOTP()`.

*   **`lib/services/fcm_token.dart`**
    *   **Purpose**: Provides a utility function specifically for **retrieving the Firebase Cloud Messaging (FCM) device token**.
    *   **Key Contents**:
        *   `getFCMToken()`: Asynchronously fetches the FCM token using `FirebaseMessaging.instance.getToken()`. It ensures this operation is performed only on mobile platforms (Android/iOS) and handles potential errors during token retrieval.

*   **`lib/services/remote_config_service.dart`**
    *   **Purpose**: Implements a singleton service for **fetching and providing dynamic configuration values** from Firebase Remote Config.
    *   **Key Contents**:
        *   Singleton Pattern: Ensures only one instance of `RemoteConfigService` is used throughout the app.
        *   `initialize()`: Configures `FirebaseRemoteConfig` settings (e.g., fetch timeout, minimum fetch interval) and then fetches and activates the latest configuration values.
        *   Getters: Provides convenient methods (`getString`, `getInt`, `getBool`, `getDouble`) to retrieve configured values by their keys, with options for default values if a key is not found.

*   **`lib/services/sim_service.dart`**
    *   **Purpose**: Acts as a **platform-specific service** to retrieve phone numbers associated with the device's SIM cards using a MethodChannel.
    *   **Key Contents**:
        *   `_channel`: A `MethodChannel` named `sim_info_channel` that facilitates communication with native Android code.
        *   `getSimNumbers()`: Invokes the native `getSimNumbers` method. It includes error handling for permission denials or if no SIM numbers are available, throwing an exception to the Dart side. (The corresponding native implementation is in `MainActivity.kt` in the Android project).
