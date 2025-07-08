# Software Requirements Specification: YouTube Video Management App (MVP)

## 1. Introduction

### 1.1 Purpose
This Software Requirements Specification (SRS) document describes the functional and non-functional requirements for the Minimum Viable Product (MVP) of the YouTube Video Management Application. This application will function as a backend API service, enabling a user to manage their YouTube videos.

### 1.2 Document Conventions
This document uses standard conventions. "Shall" is used to denote a mandatory requirement. "Should" denotes a desirable but not mandatory feature. "May" denotes an optional feature.

### 1.3 Intended Audience and Reading Suggestions
This document is intended for project stakeholders, developers, testers, and project managers. Readers should familiarize themselves with the overall project goals (Section 2) before diving into specific requirements (Sections 3, 4, 5). Familiarity with RESTful API concepts and YouTube's video management capabilities is beneficial.

### 1.4 Project Scope
The scope of this MVP is to deliver a Ruby on Rails-based API service for managing YouTube videos. Key functionalities include static user authentication, and CRUD (Create, Read, Update, Delete) operations for videos on a pre-configured YouTube channel. A frontend UI is out of scope for the MVP. The system will rely on the YouTube Data API v3.

**Reference:** See `youtube_video_manager_prd.md` for the Product Requirements Document.

### 1.5 References
*   Product Requirements Document: `youtube_video_manager_prd.md`
*   YouTube Data API v3 Documentation: [https://developers.google.com/youtube/v3/docs](https://developers.google.com/youtube/v3/docs)
*   Google API Client Library for Ruby: [https://github.com/googleapis/google-api-ruby-client](https://github.com/googleapis/google-api-ruby-client)

## 2. Overall Description

### 2.1 Product Perspective
The YouTube Video Management App is a new, standalone backend service. It will provide an API for clients (e.g., a future frontend application, scripts, or third-party services) to interact with.

### 2.2 Product Features
The major features of the MVP include:
*   Database-backed User Authentication (Sign Up, Login, Logout)
*   YouTube Account Connection Status Check (for pre-configured account)
*   Video Listing (from connected channel)
*   Viewing Video Details
*   Editing Video Metadata
*   Uploading New Videos
*   Deleting Videos

### 2.3 User Classes and Characteristics
*   **API Client (Authenticated User):** For the MVP, this is a single administrator/user who has configured their YouTube channel credentials (OAuth tokens) via environment variables. This user interacts with the system via API calls. They are expected to have technical knowledge to make API requests.

### 2.4 Operating Environment
*   **Server-side:** Ruby on Rails application.
    *   **Development Environment:** Windows
    *   Ruby version: `3.3.8 (2025-04-09 revision b200bad6cd) [x64-mingw-ucrt]`
    *   Rails version: `8.0.2`
*   Dependencies: `google-api-client` gem, `dotenv-rails` (for development), `sqlite3` gem, `solid_queue` gem.
*   **Deployment:** Deployment via Kamal. Standard Rails deployment environment (e.g., Puma server, Linux OS). Specific hosting platform TBD.
*   **Client-side (API Consumer):** Any HTTP client capable of making JSON requests (e.g., curl, Postman, custom frontend).

### 2.5 Design and Implementation Constraints
*   The application shall be developed using Ruby on Rails in API-only mode.
*   The application shall follow Clean Architecture principles.
*   User authentication for the MVP shall be database-backed using SQLite, with users having an email and a securely hashed password.
*   All interactions with YouTube shall be performed via the YouTube Data API v3 using the `google-api-client` Ruby gem.
*   Sensitive credentials (API keys, OAuth tokens, static user password) shall be managed via environment variables.
*   The API shall be RESTful and use JSON for request and response bodies.
*   Background jobs shall be processed using Solid Queue.

### 2.6 Assumptions and Dependencies
*   The YouTube Data API v3 is available and functional.
*   The user has a valid Google account and YouTube channel.
*   The user can generate and securely store OAuth 2.0 client credentials and pre-configured access/refresh tokens in environment variables for the MVP.
*   The environment variables for authentication and YouTube API access are correctly configured.
*   The API client has network connectivity to the deployed application.

## 3. System Features (Functional Requirements)

### 3.1 User Authentication
    **Feature ID:** AUTH
    **Description:** Provides mechanisms for user registration (sign up), login, and logout using database-backed credentials.

    #### 3.1.1 Sign Up
        **Req ID:** AUTH-001
        **Description:** The system shall provide an API endpoint for users to sign up.
        **Inputs:** Email, Password (in JSON request body).
        **Processing:**
            1.  Validate input parameters (email, password).
            2.  Create a new user in the database with the provided email and a securely hashed password.
        **Outputs:**
            *   Success (HTTP 201): JSON message `{"message": "User created successfully"}`.
            *   Failure (HTTP 400): JSON message `{"error": "Invalid input data"}`.

    #### 3.1.2 Login
        **Req ID:** AUTH-002
        **Description:** The system shall provide an API endpoint for users to log in.
        **Inputs:** Email, Password (in JSON request body).
        **Processing:**
            1.  Validate input parameters (email, password).
            2.  Find user by email in the database.
            3.  If user is found, verify the provided password against the user's stored hashed password.
            4.  If credentials are valid, establish a user session (e.g., set a session cookie).
        **Outputs:**
            *   Success (HTTP 200): JSON message `{"message": "Login successful"}`. Session cookie set.
            *   Failure (HTTP 401): JSON message `{"error": "Invalid email or password"}`.

    #### 3.1.3 Logout
        **Req ID:** AUTH-003
        **Description:** The system shall provide an API endpoint for users to log out.
        **Inputs:** Valid session identifier (e.g., session cookie).
        **Processing:** Invalidate/clear the user's current session.
        **Outputs:**
            *   Success (HTTP 200): JSON message `{"message": "Logout successful"}`.

### 3.2 YouTube Account Connection (MVP: Pre-configured)
    **Feature ID:** YT-CONN
    **Description:** Provides status for the pre-configured YouTube account connection.

    #### 3.2.1 Connection Status Check
        **Req ID:** YT-CONN-001
        **Description:** The system shall provide an API endpoint to check the status of the YouTube connection.
        **Inputs:** Valid session.
        **Processing:**
            1.  Verify if YouTube API credentials (tokens) are configured.
            2.  (Optional) Make a lightweight call to YouTube API to verify token validity and fetch channel name.
        **Outputs:**
            *   Success (HTTP 200): JSON `{"connected": true/false, "channel_name": "Your Channel Name" (if connected and fetched)}`.
            *   Failure (HTTP 503): JSON `{"error": "YouTube API credentials not configured or invalid"}`.

### 3.3 Video Management
    **Feature ID:** VID-MGMT
    **Description:** Provides API endpoints for managing videos on the connected YouTube channel. All endpoints require prior authentication.

    #### 3.3.1 List Videos
        **Req ID:** VID-MGMT-001
        **Description:** The system shall allow an authenticated user to list videos from their connected YouTube channel.
        **Inputs:** Valid session. Optional query parameters: `page` (for pagination), `per_page`.
        **Processing:**
            1.  Fetch the list of videos from the YouTube Data API v3 for the configured channel.
            2.  Handle pagination based on YouTube API response and input parameters.
            3.  Format video data (thumbnail, title, views, publication date) into a list.
        **Outputs:**
            *   Success (HTTP 200): JSON `{"videos": [VideoObjectShort], "pagination": { ... }}`.
            *   Failure (HTTP 500/503): JSON `{"error": "Failed to fetch videos from YouTube"}`.

    #### 3.3.2 View Video Details
        **Req ID:** VID-MGMT-002
        **Description:** The system shall allow an authenticated user to view detailed information for a specific video.
        **Inputs:** Valid session. `youtube_video_id` (path parameter).
        **Processing:** Fetch detailed video information (title, description, tags, category, privacy, stats) from YouTube Data API v3.
        **Outputs:**
            *   Success (HTTP 200): JSON `{VideoObjectFull}`.
            *   Failure (HTTP 404): JSON `{"error": "Video not found"}`.
            *   Failure (HTTP 500/503): JSON `{"error": "Failed to fetch video details"}`.

    #### 3.3.3 Edit Video Metadata
        **Req ID:** VID-MGMT-003
        **Description:** The system shall allow an authenticated user to update metadata for a specific video.
        **Inputs:** Valid session. `youtube_video_id` (path parameter). JSON request body with fields to update (title, description, tags, category_id, privacy_status).
        **Processing:** Send update request for the specified video metadata to YouTube Data API v3.
        **Outputs:**
            *   Success (HTTP 200): JSON `{VideoObjectFull (updated)}`.
            *   Failure (HTTP 400): JSON `{"error": "Invalid input data"}`.
            *   Failure (HTTP 404): JSON `{"error": "Video not found"}`.
            *   Failure (HTTP 500/503): JSON `{"error": "Failed to update video"}`.

    #### 3.3.4 Upload New Video
        **Req ID:** VID-MGMT-004
        **Description:** The system shall allow an authenticated user to upload a new video.
        **Inputs:** Valid session. `multipart/form-data` request including video file and metadata (title, description, tags, category_id, privacy_status).
        **Processing:**
            1.  Receive video file and metadata.
            2.  Upload the video file and metadata to the YouTube Data API v3.
        **Outputs:**
            *   Success (HTTP 201): JSON `{VideoObjectFull (newly uploaded)}`.
            *   Failure (HTTP 400): JSON `{"error": "Invalid input data or file issue"}`.
            *   Failure (HTTP 500/503): JSON `{"error": "Failed to upload video"}`.

    #### 3.3.5 Delete Video
        **Req ID:** VID-MGMT-005
        **Description:** The system shall allow an authenticated user to delete a specific video.
        **Inputs:** Valid session. `youtube_video_id` (path parameter).
        **Processing:** Send delete request for the specified video to YouTube Data API v3.
        **Outputs:**
            *   Success (HTTP 204 No Content).
            *   Failure (HTTP 404): JSON `{"error": "Video not found"}`.
            *   Failure (HTTP 500/503): JSON `{"error": "Failed to delete video"}`.

## 4. External Interface Requirements

### 4.1 User Interfaces
*   For the MVP, the system shall expose a RESTful API. No graphical user interface (GUI) will be developed.
*   API documentation (e.g., OpenAPI/Swagger specification) should be considered for future iterations to aid API consumers.

### 4.2 Hardware Interfaces
*   Not applicable for this software system beyond standard server hardware.

### 4.3 Software Interfaces
*   **YouTube Data API v3:** The system shall interface with the YouTube Data API v3 for all video management operations. This includes authentication (OAuth 2.0), data retrieval, and data modification.
    *   The system shall use the `google-api-client` Ruby gem for this interaction.
*   **HTTP/S Client:** API consumers will interact with the system via HTTP/S requests.

### 4.4 Communications Interfaces
*   The API shall be accessible via HTTP/S. HTTPS is mandatory for production environments to ensure secure communication.
*   Data exchange format shall be JSON.

### 4.5 Data Storage and Persistence
*   **User Accounts:** User credentials (email and securely hashed passwords) shall be stored in a SQLite database.
*   **Background Job Queue:** The `solid_queue` gem will utilize the SQLite database for managing its job queue and state.
*   **Video Data:** Video metadata is primarily fetched from the YouTube API and is not persistently stored in the application's database for the MVP, except for potential temporary caching or data related to background job processing.

## 5. Non-Functional Requirements

### 5.1 Performance Requirements
*   **NFR-001 (Response Time):** API responses for read operations (list, view details) should ideally complete within 2-3 seconds under normal load, excluding latency from the YouTube API itself.
*   **NFR-002 (Upload Time):** Video upload time is dependent on file size and network conditions, and the YouTube API. The system should handle uploads efficiently without timing out for reasonable file sizes (e.g., up to 1GB for MVP testing).

### 5.2 Data Persistence
*   **NFR-003:** User account data and background job states managed by `solid_queue` shall be persistently stored in the SQLite database, ensuring data integrity and availability across application restarts.

### 5.3 Reliability (Background Jobs)
*   **NFR-004:** Background jobs processed by `solid_queue` (e.g., for potential future features like asynchronous video processing or notifications) shall be reliable, with mechanisms for retries and error handling as provided by the queueing system.

### 5.4 Security
*   **NFR-005 (Password Hashing):** User passwords stored in the database must be securely hashed using a strong, industry-standard algorithm (e.g., bcrypt). Plain text passwords must never be stored.

### 5.5 Performance (Continued)
*   **NFR-006 (Concurrency):** The system should be able to handle a small number of concurrent API requests (e.g., 5-10 for MVP) without significant degradation in performance.

### 5.6 General Security Requirements
*   **NFR-007 (Authentication Assurance):** All API endpoints modifying data or accessing private information shall require valid session-based authentication.
*   **NFR-008 (Credential Storage):** Sensitive credentials (API keys, OAuth tokens) shall be stored securely using environment variables, not hardcoded in the source code. The application database connection string should also be managed securely.
*   **NFR-009 (Data Transmission):** All communication between the API client and the server should be over HTTPS in a production environment.
*   **NFR-010 (Input Validation):** The system shall validate all inputs from API clients to prevent common vulnerabilities (e.g., injection attacks, though less critical for an API consumed by trusted sources in MVP).
*   **NFR-011 (Error Handling):** The system shall not expose sensitive system information or stack traces in error messages to the client.

### 5.3 Reliability
*   **NFR-009 (Error Recovery):** The system should gracefully handle errors from the YouTube API (e.g., quota limits, temporary unavailability) and provide informative error messages to the client.
*   **NFR-010 (YouTube API Dependency):** The system's reliability for video operations is directly dependent on the reliability and availability of the YouTube Data API v3.

### 5.4 Availability
*   **NFR-011 (Service Availability):** The API service should aim for high availability, dependent on the chosen hosting infrastructure and the YouTube API. Specific uptime targets are TBD post-MVP.

### 5.5 Maintainability
*   **NFR-012 (Code Quality):** Code shall be well-commented, follow Ruby and Rails best practices, and adhere to the Clean Architecture principles outlined.
*   **NFR-013 (Modularity):** The system shall be designed with modular components (Entities, Use Cases, Gateways) to facilitate easier updates and maintenance.
*   **NFR-014 (Testability):** Components, especially use cases and gateways, should be designed for unit testability (though test implementation is a separate task).

### 5.6 Portability
*   **NFR-015 (Platform Independence):** As a Ruby on Rails application, it should be portable across platforms that support Ruby and Rails.

### 5.7 Usability (API Context)
*   **NFR-016 (API Design):** API endpoints shall be intuitive, consistently named, and follow RESTful principles.
*   **NFR-017 (API Documentation):** (Future consideration) Clear API documentation should be provided.
*   **NFR-018 (Error Messages):** API error messages shall be clear and help the client understand the nature of the problem.

## 6. Other Requirements

### 6.1 Data Management
*   For the MVP, the application will not have its own persistent database for application data beyond session storage. All video data is managed and stored by YouTube.

### 6.2 Deployment
*   The application shall be deployable to a standard Ruby on Rails hosting environment.
*   Configuration (environment variables) shall be manageable per deployment environment.

---
**Glossary**
*   API: Application Programming Interface
*   CRUD: Create, Read, Update, Delete
*   ENV: Environment Variables
*   GUI: Graphical User Interface
*   HTTP/S: Hypertext Transfer Protocol / Secure
*   JSON: JavaScript Object Notation
*   MVP: Minimum Viable Product
*   OAuth: Open Authorization
*   PORO: Plain Old Ruby Object
*   PRD: Product Requirements Document
*   RESTful: Representational State Transfer
*   SRS: Software Requirements Specification
