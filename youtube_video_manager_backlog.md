# Project Backlog: YouTube Video Management App (MVP)

## Sprint Goal (MVP Completion)
Deliver a functional backend API service for managing YouTube videos on a pre-configured channel. The system will feature database-backed user authentication (SQLite), core CRUD operations for videos, background job processing capabilities via Solid Queue, and utilize a Clean Architecture approach. It will allow for switching between a mock and a real YouTube API gateway.

## I. Project Setup & Initial Configuration (Foundation)

*   **TASK-001:** Initialize new Ruby on Rails application (`youtube_manager_app`) in API-only mode. [DONE]
    *   *Details:* Configure Ruby (3.3.8) and Rails (8.0.2).
    *   *Depends on:* None
*   **TASK-002:** Add and configure `dotenv-rails` for environment variable management. [DONE - Gemfile updated; manual .env.example creation requested]
    *   *Details:* Create `.env` and `.env.example` files. Add `.env` to `.gitignore`.
    *   *Depends on:* TASK-001
*   **TASK-003:** Define basic directory structure for Clean Architecture (entities, use_cases, gateways, presenters). [DONE]
    *   *Depends on:* TASK-001
*   **TASK-004:** Add `sqlite3` gem to `Gemfile` (usually present by default). Configure `config/database.yml` for SQLite. [DONE]
    *   *Depends on:* TASK-001
*   **TASK-005:** Run initial `rails db:create` and `rails db:migrate` (if any initial migrations exist). [DONE]
    *   *Depends on:* TASK-004
*   **TASK-006:** Add `solid_queue` gem to `Gemfile`. [DONE - Included by default in Rails 8.0.2]
    *   *Depends on:* TASK-001
*   **TASK-007:** Run `solid_queue` installation generator (`rails g solid_queue:install`). [DONE]
    *   *Depends on:* TASK-006
*   **TASK-008:** Run `solid_queue` migrations (`rails db:migrate`). [DONE]
    *   *Depends on:* TASK-007
*   **TASK-009:** Configure `solid_queue` adapter in `config/application.rb` or environment files (`config.active_job.queue_adapter = :solid_queue`). [DONE]
    *   *Depends on:* TASK-006

## II. Database-backed User Authentication

*   **TASK-010:** Generate `User` model (`rails g model User email:string:uniq password_digest:string`). [DONE]
    *   *Details:* Add `has_secure_password` to `app/models/user.rb`. Add validations for email (presence, uniqueness, format) and password (presence, length on create).
    *   *Depends on:* TASK-004
*   **TASK-011:** Create and run migration for the `users` table. [DONE]
    *   *Depends on:* TASK-010
*   **TASK-012:** Implement `CreateUser` use case (`app/use_cases/create_user.rb`). [DONE]
    *   *Details:* Takes email and password, creates a new User record. Handles potential errors (e.g., email already taken).
    *   *Depends on:* TASK-010
*   **TASK-013:** Implement `AuthenticateUser` use case (update existing or create new). [DONE]
    *   *Details:* Takes email and password. Finds user by email, uses `user.authenticate(password)`.
    *   *Depends on:* TASK-010
*   **TASK-014:** Implement `Api::V1::BaseController` with `require_login` and `current_user` methods for session-based auth. [DONE]
    *   *Details:* `current_user` will find user from `session[:user_id]`.
    *   *Depends on:* TASK-001, TASK-010
*   **TASK-015:** Implement `Api::V1::AuthController` with `signup`, `login`, and `logout` actions. [DONE]
    *   *Details:* `signup` uses `CreateUser`. `login` uses `AuthenticateUser` and sets `session[:user_id]`. `logout` clears `session[:user_id]`.
    *   *Depends on:* TASK-012, TASK-013, TASK-014
*   **TASK-016:** Define API routes for `/api/v1/auth/signup`, `/api/v1/auth/login`, and `/api/v1/auth/logout`. [DONE]
    *   *Depends on:* TASK-015
*   **TASK-017:** Manually test authentication endpoints (signup, login with correct/incorrect credentials, logout). [DONE - Basic integration tests implemented and run]
    *   *Depends on:* TASK-016

## III. YouTube API Gateway (Mock Implementation First)

*   **TASK-018:** Implement `MockYouTubeApiGateway` class. [DONE]
    *   *Details:* Create methods mirroring the real gateway (`list_videos`, `get_video_details`, `upload_video`, `update_video`, `delete_video`). Use in-memory data store. Return `OpenStruct` objects.
    *   *Depends on:* TASK-003
*   **TASK-019:** Populate `MockYouTubeApiGateway` with initial sample video data. [DONE]
    *   *Depends on:* TASK-018
*   **TASK-020:** Implement strategy for conditionally instantiating `MockYouTubeApiGateway` or `YouTubeApiGateway` (e.g., based on `ENV['USE_MOCK_YOUTUBE_API']`). [DONE]
    *   *Depends on:* TASK-018 (and later real gateway tasks)

## IV. Core Video Management Features (Using Mock Gateway)

### A. Entities (POROs - already designed, ensure they are created)
*   **TASK-021:** Create `Video` entity (`app/entities/video.rb`). [DONE]
    *   *Depends on:* TASK-003
*   **TASK-022:** Create `YouTubeChannel` entity (`app/entities/youtube_channel.rb`). [DONE]
    *   *Depends on:* TASK-003

### B. Use Cases & Controllers & Presenters
*   **TASK-023:** Implement `ListChannelVideos` use case. [DONE]
    *   *Depends on:* TASK-020, TASK-021
*   **TASK-024:** Implement `Api::V1::VideosController#index` action. [DONE]
    *   *Depends on:* TASK-014, TASK-023
*   **TASK-025:** Implement Presenter/Serializer for video list response. [DONE]
    *   *Depends on:* TASK-021, TASK-024
*   **TASK-026:** Implement `GetVideoDetails` use case. [DONE]
    *   *Depends on:* TASK-020, TASK-021
*   **TASK-027:** Implement `Api::V1::VideosController#show` action. [DONE]
    *   *Depends on:* TASK-014, TASK-026
*   **TASK-028:** Implement Presenter/Serializer for single video detail response. [DONE - Covered by VideoPresenter]
    *   *Depends on:* TASK-021, TASK-027
*   **TASK-029:** Implement `UploadNewVideo` use case. [DONE]
    *   *Details:* Consider if any part of this could be a background job with `solid_queue` later (e.g., post-upload processing, not the upload itself for MVP).
    *   *Depends on:* TASK-020, TASK-021
*   **TASK-030:** Implement `Api::V1::VideosController#create` action. [DONE]
    *   *Depends on:* TASK-014, TASK-029
*   **TASK-031:** Implement `UpdateVideoMetadata` use case. [DONE]
    *   *Depends on:* TASK-020, TASK-021
*   **TASK-032:** Implement `Api::V1::VideosController#update` action. [DONE]
    *   *Depends on:* TASK-014, TASK-031
*   **TASK-033:** Implement `DeleteVideo` use case. [DONE]
    *   *Depends on:* TASK-020
*   **TASK-034:** Implement `Api::V1::VideosController#destroy` action. [DONE]
    *   *Depends on:* TASK-014, TASK-033
*   **TASK-035:** Define API routes for all `/api/v1/videos` CRUD actions. [DONE]
    *   *Depends on:* Relevant controller actions.
*   **TASK-036:** Manually test all video CRUD API endpoints using the Mock Gateway. [Automated tests implemented; local verification of test execution/output needed]
    *   *Depends on:* TASK-035, TASK-017 (for auth)

## V. YouTube API Gateway (Real Implementation) & YouTube Connection Status

*   **TASK-037:** Add `google-api-client` gem to `Gemfile` and `bundle install` (if not already done). [DONE]
    *   *Depends on:* TASK-001
*   **TASK-038:** Implement `YouTubeApiGateway` class structure. [DONE]
    *   *Details:* Initialize `Google::Apis::YoutubeV3::YouTubeService` and implement `authorize` method using ENV variables.
    *   *Depends on:* TASK-003, TASK-037
*   **TASK-039:** Obtain and configure all necessary YouTube API credentials in `.env`. [DONE - .env.example guidance provided]
    *   *Depends on:* TASK-002
*   **TASK-040:** Implement `list_videos` method in `YouTubeApiGateway`. [DONE - Initial implementation]
    *   *Depends on:* TASK-038, TASK-039
*   **TASK-041:** Implement `get_video_details` method in `YouTubeApiGateway`. [DONE - Initial implementation]
    *   *Depends on:* TASK-038, TASK-039
*   **TASK-042:** Implement `upload_video` method in `YouTubeApiGateway`. [DONE - Initial implementation]
    *   *Depends on:* TASK-038, TASK-039
*   **TASK-043:** Implement `update_video` method in `YouTubeApiGateway`. [DONE - Initial implementation]
    *   *Depends on:* TASK-038, TASK-039
*   **TASK-044:** Implement `delete_video` method in `YouTubeApiGateway`. [DONE - Initial implementation]
    *   *Depends on:* TASK-038, TASK-039
*   **TASK-045:** Implement `Api::V1::YoutubeConnectionController` with `status` action. [DONE]
    *   *Depends on:* TASK-014, TASK-038
*   **TASK-046:** Define API route for `/api/v1/youtube/connection_status`. [DONE]
    *   *Depends on:* TASK-045
*   **TASK-047:** Manually test YouTube connection status endpoint.
    *   *Depends on:* TASK-046, TASK-039, TASK-017 (for auth)

## VI. Integration & Testing (Using Real Gateway)

*   **TASK-048:** Switch gateway instantiation to use `YouTubeApiGateway`.
    *   *Depends on:* All tasks in Section IV & V.
*   **TASK-049:** Manually test all video CRUD API endpoints using the real `YouTubeApiGateway`.
    *   *Depends on:* TASK-048, TASK-017 (for auth)
*   **TASK-050:** Implement basic error handling in `Api::V1::BaseController` using `rescue_from`.
    *   *Depends on:* TASK-014

## VII. Documentation & Final Review (MVP)

*   **TASK-051:** Review and update PRD and SRS documents if any deviations occurred. (Already done for DB/SolidQueue shift)
*   **TASK-052:** Create a simple Postman collection or cURL command list for testing API endpoints.
*   **TASK-053:** Code review and refactoring.

---
**Future Considerations (Post-MVP - Not in this backlog)**
*   Full OAuth 2.0 Flow for dynamic YouTube account linking.
*   Multi-User YouTube Channel Management (if users can link multiple channels).
*   Advanced Video Management Features (batch operations, analytics display).
*   Webhooks for real-time updates.
*   Comprehensive Automated Testing (Unit, Integration for Use Cases, Controllers, Gateways).
*   Asynchronous video uploads/processing using `solid_queue`.
*   Frontend UI.
