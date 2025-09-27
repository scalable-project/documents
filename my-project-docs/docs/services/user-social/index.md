# User & Social Service (USS) Overview

The User & Social Service (USS) is a core microservice within the Campus Connect application. It serves as the authoritative source for all user-related data, managing user identities, authentication, profiles, and the social graph (following/followers). It is designed to be highly secure and reliable, ensuring the integrity of sensitive user information.

---

## 1. Core Responsibilities

The USS adheres strictly to the Single Responsibility Principle, focusing on the following key business domains:

- **Identity Management:** Handles user registration, login, and secure storage of user credentials.
- **Authentication & Authorization:** Validates user identity during login and manages JWT token generation for subsequent authenticated requests via the API Gateway.
- **Profile Management:** Stores and serves all public and private user profile data (e.g., username, email, bio).
- **Social Graph Management:** Manages the following/follower relationships between students and organizations.
- **Event Producer:** Acts as an event producer by publishing user interaction events to Kafka, enabling other services (like the Real-time Notification Service) to react.

---

## 2. Technology Stack

| **Component**       | **Technology**           | **Rationale**                                                                 |
|----------------------|--------------------------|-------------------------------------------------------------------------------|
| Language/Framework   | Not Decided yet   | NA                      |
| Database             | PostgreSQL              | Provides strong transactional integrity and reliable schema for user data.   |
| Communication        | REST API (synchronous), Apache Kafka (asynchronous) | REST for immediate profile/login needs; Kafka for broadcasting social actions. |

---

## 3. API Summary

The USS exposes secure REST endpoints, primarily consumed by the API Gateway.

| **Endpoint**          | **Method** | **Function**                              | **Key Data Handled**                                   |
|------------------------|-----------|------------------------------------------|-------------------------------------------------------|
| `/users/register`      | POST      | Creates a new user account.              | Username, Email, Password.                           |
| `/users/login`         | POST      | Authenticates a user and returns a JWT.  | Email, Password.                                     |
| `/users/{id}`          | GET       | Retrieves a user's public profile data.  | Username, Bio, Follower/Following Counts.            |
| `/users/{id}/follow`   | POST      | Creates a one-way follow relationship.   | `follower_id` (authenticated user), `target_user_id`. |
| `/users/{id}/followers`| GET       | Returns a list of users following the ID.| List of User IDs.                                    |

---

## 4. Data Model (PostgreSQL)

The service maintains a dedicated PostgreSQL database with two main tables to define its domain:

### 4.1. `users` Table

The source of truth for user identity.

| **Column**            | **Type**    | **Description**                                      |
|------------------------|------------|-----------------------------------------------------|
| `user_id`             | UUID       | Primary Key.                                        |
| `email`               | VARCHAR    | Unique, NOT NULL (used for login).                 |
| `password_hash`       | VARCHAR    | NOT NULL (hashed password).                        |
| `username`            | VARCHAR    | Unique, NOT NULL (public identifier).              |
| `is_organization`     | BOOLEAN    | Differentiates between a student and an organization. |
| `bio`                 | TEXT       | User bio.                                          |
| `profile_picture_url` | TEXT       | URL to the user's profile picture.                 |
| `created_at`          | TIMESTAMP  | Metadata for record creation.                      |

---

### 4.2. `followers` Table (Junction Table)

Manages the many-to-many relationship of the social graph.

| **Column**      | **Type** | **Description**                                      |
|------------------|----------|-----------------------------------------------------|
| `follower_id`   | UUID     | Foreign Key linking to the user who initiated the follow. |
| `following_id`  | UUID     | Foreign Key linking to the user/organization being followed. |
| **Primary Key** | Composite key of (`follower_id`, `following_id`) to ensure no duplicate relationships. |

---

## 5. Event Production (Asynchronous Output)

When a social action occurs, the USS publishes an event to Kafka to notify other services:

| **Action**                     | **Event Type**       | **Kafka Topic**   | **Recipient Services**                                   |
|--------------------------------|----------------------|-------------------|---------------------------------------------------------|
| User successfully follows another. | `user_followed`     | `user_events`     | Real-time Notification Service, Discovery & Recommendation Service. |
| User unfollows another.         | `user_unfollowed`   | `user_events`     | Discovery & Recommendation Service.                    |