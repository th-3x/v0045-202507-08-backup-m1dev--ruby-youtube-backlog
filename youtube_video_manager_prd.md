# Product Requirements Document: YouTube Video Management App (MVP)

## 1. Introduction

This document outlines the requirements for the Minimum Viable Product (MVP) of a YouTube Video Management Application. The primary goal is to provide a backend API service, built with Ruby on Rails, allowing a user to manage their YouTube videos. Initial focus is on core video management functionalities with a database-backed user authentication mechanism and background job processing.

## 2. Goals and Objectives

*   **Primary Goal:** Enable a user to perform basic CRUD (Create, Read, Update, Delete) operations on their YouTube videos via a secure API.
*   **MVP Objective 1:** Implement a stable API for listing, viewing details, editing metadata, uploading, and deleting YouTube videos.
*   **Data Storage:** A SQLite database will be used for storing user accounts and for the `solid_queue` background job processing system. Video metadata will primarily be fetched from the YouTube API on demand for the MVP and not stored persistently in our database, unless cached by `solid_queue` or future features.
*   **MVP Objective 3:** Establish a clean architectural foundation (Clean Architecture on Rails) for future scalability and maintainability.
*   **MVP Objective 4:** Integrate with the YouTube Data API v3 for all video operations.

## 3. Target Audience

*   **MVP:** A single administrator/user who will pre-configure their YouTube account credentials (via OAuth tokens stored in environment variables) to manage their own YouTube channel.

## 4. Proposed Features (MVP)

### 4.1. User Authentication (Database-backed)
*   **Database-backed Authentication:** User authentication will be managed via a SQLite database. Users will have accounts with email and password (securely hashed). This provides a more robust and scalable authentication mechanism than static credentials.
*   **4.1.2. Logout System:** API endpoint to invalidate the current session.
*   **4.1.3. Access Control:** All video management API endpoints will require prior authentication.

### 4.2. YouTube Account Management (MVP: Pre-configured)
*   **4.2.1. Connect to YouTube:** The application will use pre-configured OAuth 2.0 credentials (client ID, client secret, access token, refresh token, channel ID) stored in environment variables to interact with a specific YouTube channel.
*   **4.2.2. Connection Status:** API endpoint to check if the application is configured to connect to YouTube and display basic channel info (e.g., channel name).

### 4.3. Core Video Management Features (API Endpoints)
*   **4.3.1. List Videos:**
    *   Fetch and display a paginated list of videos from the connected YouTube channel.
    *   Key details per video: Thumbnail, Title, Views, Publication Date.
*   **4.3.2. View Video Details:**
    *   Fetch and display comprehensive details for a selected video: Title, Description, Tags, Category, Privacy Status, Statistics (Likes, Comments).
*   **4.3.3. Edit Video Metadata:**
    *   Update a video's Title, Description, Tags, Category, and Privacy Status on YouTube.
*   **4.3.4. Upload New Video:**
    *   Upload a video file along with its initial metadata (Title, Description, Tags, Category, Privacy Status) to the connected YouTube channel.
*   **4.3.5. Delete Video:**
    *   Delete a video from the YouTube channel. A confirmation step (if a UI were present) would be implicit in the API call.

## 5. API Design (Summary - RESTful JSON)

*   **Authentication:**
    *   `POST /api/v1/auth/login`
    *   `DELETE /api/v1/auth/logout`
*   **YouTube Connection:**
    *   `GET /api/v1/youtube/connection_status`
*   **Video Management (Protected):**
    *   `GET /api/v1/videos` (List with pagination)
    *   `GET /api/v1/videos/:youtube_video_id` (View details)
    *   `PUT /api/v1/videos/:youtube_video_id` (Update metadata)
    *   `POST /api/v1/videos` (Upload new video)
    *   `DELETE /api/v1/videos/:youtube_video_id` (Delete video)

## 6. Technical Architecture (Summary)

*   **Framework:** Ruby on Rails (API-only mode).
*   **Architecture:** Clean Architecture principles.
    *   **Entities:** Plain Ruby Objects (e.g., `Video`, `YouTubeChannel`).
    *   **Use Cases:** Plain Ruby Objects for application-specific business logic.
    *   **Interface Adapters:**
        *   Rails Controllers (for API request/response handling).
        *   Presenters/Serializers (for JSON formatting).
        *   Gateways (`YouTubeApiGateway` for YouTube API interaction, `AuthenticationGateway` for database-backed auth).
    *   **Frameworks & Drivers:** Rails, `google-api-client` gem for YouTube Data API v3, `sqlite3` gem for SQLite database interaction, `solid_queue` gem for background job processing.
*   **Authentication:** Session-based after initial database-backed credential check.
*   **Configuration:** Heavy reliance on environment variables for credentials and API keys, managed with `dotenv-rails` in development.

## 7. Success Metrics (MVP)

*   All defined API endpoints are functional and operate as expected.
*   User can successfully authenticate and de-authenticate.
*   User can list, view, edit, upload, and delete videos on their connected YouTube channel via the API.
*   The application maintains stability during these operations.
*   Codebase adheres to the defined Clean Architecture principles.

## 8. Future Considerations (Post-MVP)

*   **Full OAuth 2.0 Flow:** Implement the complete OAuth 2.0 authorization code grant flow to allow users to connect their YouTube accounts dynamically and securely, including token refresh mechanisms.
*   **Multi-User Support:** Allow multiple users to register and manage their respective YouTube channels.
*   **Database Integration:** Store user information, YouTube channel connections, and potentially cached video data in a persistent database (e.g., PostgreSQL).
*   **Background Job Processing:** Offload time-consuming tasks like video uploads and batch updates to background jobs (e.g., using Sidekiq or GoodJob).
*   **Frontend UI:** Develop a web interface for users to interact with the application.
*   **Advanced Video Management:** Features like batch editing, playlist management, comment moderation, analytics display.
*   **Enhanced Search/Filtering:** More sophisticated video searching and filtering capabilities.
*   **Webhooks:** Listen to YouTube API webhooks for real-time updates.

## 9. Out of Scope for MVP

*   Frontend User Interface.
*   Full, dynamic OAuth 2.0 flow for user-initiated channel connections.
*   Support for multiple user accounts.
*   Database persistence for application data (beyond session).
*   Background job processing.
*   Advanced video analytics or playlist management.
*   Real-time features or webhooks.
