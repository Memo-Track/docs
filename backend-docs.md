The backend directory houses the core Django application for MemoTrack-SAAS, designed for various hospitals. It includes several Django apps, configuration files, and static content.
Here's a breakdown of each file and its purpose:
• backend.zip: This is a binary file, likely a compressed archive of the backend code.
• data.json: This is a binary file, suggesting it contains data used by the backend, possibly for initial setup or specific functionalities.
• manage.py:
    ◦ This is Django's command-line utility for administrative tasks.
    ◦ It sets the default port for the runserver command to 30000.
    ◦ It ensures the DJANGO_SETTINGS_MODULE environment variable is set to core.settings before executing commands.
• requirements.txt: This is a binary file, which would typically list the Python dependencies required for the backend application.
This Django app handles user accounts, roles, and hospital management.
• __init__.py: An empty file that signifies the accounts directory is a Python package.
• admin.py:
    ◦ Configures Django Admin to display User and Hospital models.
    ◦ HospitalAdmin: Controls the queryset for hospitals in the admin panel; superusers can see all hospitals, while other users only see hospitals they are associated with.
    ◦ UserAdmin: Controls the queryset for users; superusers see all users, while other users only see users belonging to their hospital.
• apps.py:
    ◦ Defines the AccountsConfig for the accounts app.
    ◦ It sets default_auto_field to django.db.models.BigAutoField.
    ◦ The ready method ensures that signal handlers defined in accounts.signals are imported and registered when the app is ready.
• auth_backends.py: An empty file, which could be used for custom authentication backends.
• fcm_service.py:
    ◦ Handles Firebase Cloud Messaging (FCM) functionalities.
    ◦ Initialisation: It initialises the Firebase Admin SDK using a service account key located at memo-track-firebase-adminsdk-fbsvc-060fb4e2e6.json.
    ◦ send_fcm_to_user(token, title, body, data): Sends a single FCM notification to a specific device token.
    ◦ send_fcm_to_users(tokens, title, body, data): Sends FCM notifications to a list of device tokens, returning success/failure counts and a list of failed tokens.
    ◦ memo_created_notification(info, memoId): Sends a notification when a new memo is created, targeting approvers and responders based on current block and tagged role, respectively.
    ◦ memo_tagged_notification(info, tagged_role_name, tagged_role_id, memoId): Sends a notification when a memo is tagged to a specific role.
• memo-track-firebase-adminsdk-fbsvc-060fb4e2e6.json:
    ◦ This JSON file contains the Firebase service account credentials for the "memo-track" project.
    ◦ It includes the project ID (memo-track), private key ID, the private key itself, client email, client ID, and various Google API URLs for authentication and token exchange.
• models.py:
    ◦ Role:
        ▪ Defines user roles within a Hospital.
        ▪ Fields include name, description, is_protected (prevents deletion), is_approver, is_responder, and is_creator.
        ▪ Roles are unique by name and hospital.
    ◦ Hospital:
        ▪ Represents a hospital entity.
        ▪ Fields include name, address, geolocation_point (as latitude, longitude), radius (in meters), created_at, updated_at.
        ▪ It tracks geolocation_changes when the geolocation_point is updated.
    ◦ UserManager: A custom manager for the User model, used for creating regular users and superusers.
        ▪ create_user: Creates a user with institution_id, role, phone_number, and hospital. Automatically sets is_staff=True if the role is 'ADMIN'.
        ▪ create_superuser: Creates a superuser, setting is_staff=True and is_superuser=True.
    ◦ User:
        ▪ A custom user model based on Django's AbstractBaseUser and PermissionsMixin.
        ▪ Key fields include institution_id (display ID unique within a hospital), hospital, role, phone_number (unique and validated with a regex for Indian phone numbers), current_block, current_ward, fcm_token and fcm_token_updated_at for Firebase Cloud Messaging, current_device, is_active, is_logged_in, is_staff, is_superuser, date_joined, and password_set status.
        ▪ Initially, institution_id and hospital were unique together, but this constraint was removed.
    ◦ SiteUser:
        ▪ Represents administrators for the entire MemoTrack system (not hospital-specific).
        ▪ Stores name and password.
        ▪ The save method automatically hashes the password using SHA256.
        ▪ check_password verifies a raw password against the stored hash.
• serializers.py:
    ◦ HospitalSerializer: Serialises Hospital model data, including id, name, geolocation_point, radius, and address.
    ◦ AuthTokenRequestSerializer: Used for login requests, requiring phone and password, with an optional fcm_token.
    ◦ UserSerializer: Serialises the User model, providing read-only fields for current_block_name, current_block_id, current_ward_id, current_ward_name, floor, role name and ID, hospital_id, and role-based permissions (is_approver, is_responder, is_creator, is_superuser, is_staff).
    ◦ AuthTokenResponseSerializer: Used for login responses, providing the authentication token, and the user and hospital data serialised by their respective serializers.
• signals.py:
    ◦ update_admin_staff_status: A post_save signal receiver for the User model.
    ◦ If a user's role name is 'ADMIN', it updates all users with that role to have is_staff=True. If a user's role is not 'ADMIN' and they are staff (but not superuser), their is_staff status is set to False.
• tests.py: An empty file, typically reserved for Django unit tests for the accounts app.
• urls.py:
    ◦ Defines URL patterns for administrative login, logout, dashboard, and management pages.
    ◦ Includes endpoints for managing SiteUser accounts, Hospital records, and User accounts within hospitals.
• views.py:
    ◦ Contains Django views for rendering HTML pages and Django REST Framework APIView classes for API endpoints.
    ◦ Admin HTML Views: admin_login, admin_logout, admin_dashboard, admin_page (for managing site-wide admins), delete_siteuser, admin_profile, hospitals (for managing hospitals), delete_hospital, switch_hospital (to switch admin context), hospital_users (for managing users within a specific hospital), and delete_hospital_user.
    ◦ CustomAuthToken (APIView): Handles user authentication with phone number and password. It retrieves or creates an auth token, updates the user's is_logged_in status, FCM token, and last_login timestamp.
    ◦ LogoutView (APIView): Logs out the authenticated user. For superusers, it removes their hospital association, while for others, it sets is_logged_in to False.
    ◦ RefreshUserView (APIView): Allows an authenticated user to refresh their user data and token.
This directory contains custom Django management commands for data setup.
• setup_nurses.py: A command to seed nurse users into the database from a specified JSON file. It creates a "Ward Staff Nurse" role if it doesn't exist and associates users with the first hospital found.
• setup_users.py: A command to seed a predefined set of roles (e.g., 'Ward Staff Nurse', various 'Approvers', and 'Responders') and users from an embedded JSON string for the first hospital in the system.
• setup_users_from_json.py: Similar to setup_nurses.py, but designed to process a JSON file containing nested department data to create nurse users.
This directory contains Django migration files, which track changes to the accounts app's database schema over time. Examples include initial model creation, adding/removing fields, and altering unique constraints.
This directory contains HTML templates specifically for the accounts Django app's web interface.
• admin_dashboard.html: The main dashboard for administrators, providing links to manage site admins, hospitals, and profiles.
• admin_login.html: The login form for system administrators.
• admin_page.html: A page for managing (adding/deleting) site-wide admin users.
• admin_profile.html: A page for administrators to update their profile (username and password).
• hospital_users.html: A page to view and manage users associated with a specific hospital, allowing addition and deletion of users.
• hospitals.html: A page to manage (add/delete) hospitals and navigate to manage users within each hospital.
This directory contains sub-applications related to core functionalities like infrastructure and memos.
• __init__.py: An empty file that signifies the app directory is a Python package.
This Django app manages hospital infrastructure entities like blocks, wards, and approval hierarchies.
• __init__.py: An empty file.
• admin.py: An empty file, intended for Django Admin configurations specific to infrastructure models.
• apps.py: Defines the InfrastructureConfig for the app.infrastructure app.
• models.py:
    ◦ ApproverHierarchy: Defines a sequence of approval levels for memos, indicating if the hierarchy is is_active.
    ◦ ApproverLevel: Represents a specific Role within an ApproverHierarchy and its priority (lower means higher priority).
    ◦ BlockChange: Records when a user changes their current_block, including from_block, to_block, user, and changed_at.
    ◦ WardChange: Records when a user changes their current_ward, including from_ward, to_ward, user, and changed_at.
• permissions.py: Defines custom Django REST Framework permission classes to control access to API endpoints based on user roles and hospital affiliation.
    ◦ HospitalPermission: Allows safe methods (GET, HEAD, OPTIONS) for all. PUT and PATCH are restricted to superuser or staff. DELETE is never allowed.
    ◦ IsHospitalUserOrSuperUser: Allows safe methods for all users. POST is restricted to superuser. PUT/PATCH and DELETE are allowed for superuser or staff modifying their own hospital.
    ◦ RoleEditPermission: Allows safe methods for all. All methods are allowed for superuser. staff can only perform actions on roles within their own hospital.
    ◦ UserPermission: Allows safe methods for all. All methods are allowed for superuser. staff can add/edit/delete users within their own hospital. A normal user can PUT/PATCH their own profile.
    ◦ WardPermission: Allows safe methods for all. POST and DELETE are allowed for superuser or staff within their own hospital.
• serializers.py:
    ◦ HospitalSerializer: (Redundant definition, already in accounts/serializers.py) Serialises Hospital model data.
    ◦ RoleSerializer: Serialises Role objects, showing the hospital name, and is_approver, is_responder, is_creator, is_protected flags.
    ◦ HospitalUserSerializer: Serialises User objects for hospital-specific contexts. It handles creation of users by associating them with a Hospital and setting their password.
    ◦ BlockSerializer: Serialises Blocks model data. It automatically associates new blocks with the hospital_id from the URL context and sets the created_by user.
    ◦ ShiftSerializer: Serialises Shift model data. Similar to BlockSerializer, it handles hospital association and created_by user.
    ◦ WardSerializer: Serialises Ward model data, including its associated block and floor. It includes a validate method to ensure the floor is within the block's no_of_floors range.
    ◦ ApproverLevelSerializer: Serialises ApproverLevel objects, exposing role, role_name, and priority.
    ◦ ApproverHierarchySerializer: Serialises ApproverHierarchy objects, including nested ApproverLevelSerializer for its levels. It requires at least one level and handles creation and updates of nested levels.
• tests.py: An empty file for infrastructure-related tests.
• urls.py:
    ◦ Defines API routes for hospital-related infrastructure using rest_framework.routers.DefaultRouter and rest_framework_nested.routers.NestedDefaultRouter.
    ◦ Includes nested routes under /api/hospitals/ for roles, users, shifts, blocks, hierarchies, and wards.
    ◦ Also defines direct paths for /api/set-password/, /api/switch-block/, and /api/switch-ward/.
• views.py:
    ◦ Contains viewsets.ModelViewSet classes for various infrastructure models and APIView classes for specific actions.
    ◦ HospitalViewSet: Manages Hospital records, using IsHospitalUserOrSuperUser permissions. It explicitly disallows DELETE operations.
    ◦ PasswordChangeViewSet: An APIView to allow users to change their password by providing their phone number and new password.
    ◦ RoleViewSet: Manages Role records within a hospital. It applies RoleEditPermission and allows filtering roles by their is_responder status.
    ◦ HospitalUserViewSet: Manages User records specifically for a given hospital, excluding superusers. It uses UserPermission.
    ◦ BlockViewSet: Manages Blocks within a hospital. It associates the creating user and hospital with the new block.
    ◦ ShiftViewSet: Manages Shift records within a hospital, associating the creating user and hospital. It explicitly disallows DELETE operations.
    ◦ ApproverHierarchyViewSet: Manages ApproverHierarchy records, filtering by roles within a hospital.
    ◦ WardViewSet: Manages Ward records. It allows filtering by block and floor and supports soft-deletion (marking as deleted=True) and restoring of wards, with permanent deletion only for superusers.
    ◦ SwitchBlock (APIView): Allows a user to change their current_block and logs this change as a BlockChange event.
    ◦ SwitchWard (APIView): Allows a user to change their current_ward, logs it as a WardChange event, and automatically updates their current_block to the ward's associated block.
This directory contains Django migration files for the app.infrastructure app, tracking database schema changes related to ApproverHierarchy, ApproverLevel, BlockChange, and WardChange models.
This Django app manages memo creation, events, snapshots, and dashboard data.
• __init__.py: An empty file.
• admin.py: An empty file, intended for Django Admin configurations specific to memo models.
• apps.py: Defines the MemosConfig for the app.memos app. The ready method ensures signals are imported.
• cron_job.py: Contains functions for precomputing dashboard data for performance.
    ◦ get_card_metrics(data): Calculates overall memo statistics like total, completed, completion rate, average resolution time, and fastest responder.
    ◦ get_block_ward_heatmap(data): Generates counts of memos per block and ward for heatmaps.
    ◦ get_responder_resolution(data): Calculates the average resolution time for each responder role.
    ◦ get_status_pie(data): Categorises memos into 'completed', 'rejected', and 'ongoing' statuses.
    ◦ get_approval_flow(data): Tracks the approval status of memos across predefined hierarchy levels (e.g., Ward Superintendent, RMO, DEAN).
    ◦ get_department_distribution(data): Counts memos per department.
    ◦ get_completion_timeline(data): Categorizes memo completion times into buckets (e.g., 0-2h, 2-6h, Over 6h).
    ◦ precompute_dashboard_data_and_save(hospital_id=None): Generates and saves dashboard data as JSON files (e.g., hospital_id_YYYY-MM-DD.json) for specific hospitals or all combined, for 'today', 'week', and 'month' periods. Files are stored in settings.MEDIA_ROOT/dashboard/.
    ◦ precompute_all_hospitals_dashboard_data(): Triggers the precomputation process for all hospitals in the system.
    ◦ get_hospital_dashboard_data(hospital_id, period): Retrieves the precomputed dashboard data from the saved JSON files for a given hospital and period (today, week, or month).
• models.py:
    ◦ Memo: The central model for a memo, identified by a memoId (UUID). It tracks hospital association, is_deleted, is_merged, created_at, and modified_at. It has a latest_snapshot method to retrieve the most recent state.
    ◦ MemoEvents: Records every significant event related to a Memo (e.g., 'created', 'updated', 'deleted', 'approved', 'rejected', 'attended', 'completed', 'tagged'). Each event is associated with a memo, event_by user, and includes payload and metadata JSON fields.
    ◦ MemoSnapshot: Stores a immutable snapshot of a memo's state at a particular timestamp. It includes info (contextual data about the memo and user), hierarchy (approval structure), approval_history, worker_status (list of work updates), attendee_otp, and completion_otp. It has methods refresh_otp and generate_otp to create one-time passwords.
    ◦ AttendeeETA: Tracks the estimated time of arrival (eta) for a User (attendee) regarding a specific Memo. It also records created_at, updated_at, and updated_count for ETA changes.
• serializers.py:
    ◦ MemoSerializer: Serialises MemoSnapshot data to provide a summary view, showing memoId, created_at, and latest_snapshot info.
    ◦ MemoCreateSerializer: Used to create a new Memo instance and log its initial 'created' event. It also includes a delete method for soft-deleting memos.
    ◦ MemoSnapshotSerializer: A comprehensive serializer for MemoSnapshot, exposing all its fields including OTPs.
    ◦ AttendeeETASerializer: Serialises AttendeeETA instances, showing attendee ID and name, memo ID, eta, and tracking fields.
    ◦ MemoEventsSerializer: Serialises MemoEvents instances.
• signals.py:
    ◦ create_memo_snapshot: A post_save signal receiver for MemoEvents.
    ◦ It creates a new MemoSnapshot for most event types, capturing the memo's state and user context at that moment.
    ◦ Special handling for events:
        ▪ worker_status_delete: Updates the latest snapshot to mark a specific worker status entry as deleted.
        ▪ updated: Modifies the info field of the latest existing snapshot instead of creating a new one.
        ▪ approved / rejected: Updates the hierarchy and approval_history in the latest snapshot to reflect the approval/rejection status by the specific role.
        ▪ attended / completed / incomplete / tagged: Appends new entries to the worker_status list in the latest snapshot. If the event is completed, it also updates the memo's overall status in the info field. tagged events trigger memo_tagged_notification.
    ◦ For created events, it populates the snapshot info with user context (role, block, ward, floor) and memo details (complaint, department, tagged role). It also applies the currently is_active ApproverHierarchy to the snapshot. memo_created_notification is triggered for new memos.
• tests.py: An empty file for memo-related tests.
• urls.py: Defines URL patterns for memo API endpoints, including:
    ◦ create-memo/.
    ◦ memos/ (list user memos).
    ◦ memos/<memoId>/ (memo detail and update).
    ◦ memos/<memoId>/approve/ and reject/.
    ◦ memos/<memoId>/work-status/ (for worker updates and deletion).
    ◦ memos/<memoId>/attendee-eta/ (for ETA management).
    ◦ memos/<memoId>/refresh-otp/ (to generate new OTPs).
• views.py:
    ◦ CreateMemoView: Handles the creation of new memos and their initial MemoSnapshot, logging a 'created' event.
    ◦ ListUserMemosView: Provides structured lists of memos based on the requesting user's role (creator, approver, responder, or admin), fetching the latest snapshots for each memo and categorising them accordingly (e.g., 'My Memos', 'Under Review', 'Open Memos', 'Completed', 'Deleted Memos').
    ◦ MemoDetailView: Retrieves the latest snapshot of a specific memo. It also handles PATCH requests to update a memo's details by logging an 'updated' event.
    ◦ MemoApproveView: Allows an authenticated user to approve a memo by logging an 'approved' event.
    ◦ MemoRejectView: Allows an authenticated user to reject a memo by logging a 'rejected' event.
    ◦ AttendeeETACreateView: Provides API endpoints to view all existing ETAs for a memo, create a new ETA, or update the current user's ETA for a memo.
    ◦ MemoWorkStatusView: Handles POST requests for adding worker status updates (e.g., 'attended', 'completed', 'incomplete', 'tagged') to a memo. It includes OTP verification for 'attended', 'incomplete', 'tagged', and 'completed' statuses. It also handles DELETE requests to soft-delete a specific worker status entry by logging a worker_status_delete event.
    ◦ RefreshOTPView: Allows refreshing the attendee_otp and completion_otp for a memo's latest snapshot.
This directory contains custom Django management commands for memo-related tasks.
• generate_dashboard_data.py: A management command that triggers the precompute_all_hospitals_dashboard_data function from cron_job.py to generate precomputed dashboard data files.
This directory contains Django migration files for the app.memos app, tracking database schema changes related to Memo, MemoEvents, MemoSnapshot, and AttendeeETA models.
This directory contains the core Django project configuration.
• __init__.py: An empty file.
• asgi.py: The ASGI (Asynchronous Server Gateway Interface) configuration for the project, used for asynchronous Python web servers.
• settings.py: The main Django project settings file.
    ◦ BASE_DIR: Defines the base directory of the project.
    ◦ SECRET_KEY: A unique key for cryptographic signing.
    ◦ DEBUG: Set to True for development.
    ◦ ALLOWED_HOSTS: Specifies hostnames the Django project can serve, including '*' for development and specific production URLs.
    ◦ AUTHENTICATION_BACKENDS: Lists authentication backends, including Django's ModelBackend.
    ◦ INSTALLED_APPS: Lists all active Django applications, including accounts, management, app.infrastructure, app.memos, and various Django/DRF/CORS related apps.
    ◦ AUTH_USER_MODEL: Specifies the custom user model (accounts.User).
    ◦ REST_FRAMEWORK: Configures Django REST Framework, including default schema class, authentication classes (TokenAuthentication, SessionAuthentication), permission classes (IsAuthenticated), and filter backends.
    ◦ MIDDLEWARE: Defines the order of middleware components, including CorsMiddleware and custom OriginLoggerMiddleware.
    ◦ ROOT_URLCONF: Points to the main URL configuration file (core.urls).
    ◦ TEMPLATES: Configures Django's templating engine, including DIRS for shared templates and APP_DIRS for app-specific templates.
    ◦ WSGI_APPLICATION: The WSGI (Web Server Gateway Interface) configuration for the project.
    ◦ DATABASES: Configures the database connection, currently set up for MySQL using environment variables for credentials.
    ◦ AUTH_PASSWORD_VALIDATORS: Defines password validation rules.
    ◦ TIME_ZONE: Set to Asia/Kolkata.
    ◦ STATIC_URL: URL prefix for static files.
    ◦ SPECTACULAR_SETTINGS: Configures DRF Spectacular for API documentation generation.
    ◦ CORS Settings: CSRF_TRUSTED_ORIGINS, CORS_ALLOWED_ORIGINS, CORS_ALLOW_CREDENTIALS, CORS_ALLOW_ALL_ORIGINS, CORS_ALLOW_HEADERS, CORS_ALLOWED_METHODS are configured to handle cross-origin requests, including specific domains for frontend and development purposes.
    ◦ MEDIA_URL and MEDIA_ROOT: Define settings for serving user-uploaded media files.
• urls.py:
    ◦ The main URL configuration for the entire Django project.
    ◦ Includes URLs for Django Admin, accounts app, app.infrastructure app, and app.memos app.
    ◦ Includes custom API authentication and user refresh endpoints.
    ◦ Integrates DRF Spectacular for API schema, Swagger UI, and ReDoc documentation interfaces.
    ◦ Serves media files during development.
• wsgi.py: The WSGI configuration for the project, used for synchronous Python web servers.
This directory contains custom Django middleware.
• __init__.py: An empty file.
• origin_logger.py: Defines OriginLoggerMiddleware, a custom middleware that logs the HTTP_ORIGIN, HTTP_REFERER, and HTTP_USER_AGENT headers for every incoming request.
This Django app is for managing general hospital-related data like blocks and shifts.
• __init__.py: An empty file.
• admin.py: Configures Django Admin to display Blocks and Shift models, ensuring that non-superuser admins only see data relevant to their hospital.
• apps.py: Defines the ManagementConfig for the management app.
• models.py:
    ◦ Blocks:
        ▪ Represents a building block or section within a hospital.
        ▪ Fields include name, no_of_floors, hospital, created_by, created_at, and updated_at.
        ▪ It enforces uniqueness of name within each hospital.
    ◦ Ward:
        ▪ Represents a ward, associated with a Blocks instance.
        ▪ Fields include ward_name, block, floor, deleted (for soft-deletion), created_at, created_by, and updated_at.
        ▪ The clean method provides validation to ensure the floor number is valid for the associated block.
    ◦ Shift:
        ▪ Represents work shifts within a hospital.
        ▪ Fields include name, start_time, end_time, hospital, created_by, created_at, and updated_at.
        ▪ It enforces uniqueness of name within each hospital.
• tests.py: An empty file for management-related tests.
• views.py: An empty file, intended for management-related views.
This directory contains Django migration files for the management app, tracking database schema changes related to Blocks, Shift, and Ward models.
This directory contains general HTML templates used across the Django project.
• base.html: A base HTML template that provides a common structure for other pages, including a header with "Memotrack" branding and navigation links (Dashboard, Site Admins, Hospitals, Profile, Logout).
• error.html: A template designed to display error messages, with a "Go Back" button.
• accounts/password_setup.html: An empty file, suggesting it might be a placeholder for a password setup/reset page.
