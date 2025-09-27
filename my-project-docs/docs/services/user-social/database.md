# Database Schema & Data Flow

## 1. Database Schema (PostgreSQL)

We'll use a relational database with two main tables to manage users and their connections.

### `users` Table
This table stores all core user and organization data.

| Column Name         | Data Type      | Constraints                     | Description                                  |
| ------------------- | -------------- | ------------------------------- | -------------------------------------------- |
| `user_id`           | `UUID`         | `PRIMARY KEY`, `NOT NULL`       | Unique identifier for each user.             |
| `username`          | `VARCHAR(50)`  | `UNIQUE`, `NOT NULL`            | Public username.                             |
| `email`             | `VARCHAR(255)` | `UNIQUE`, `NOT NULL`            | User's email for login/recovery.             |
| `password_hash`     | `VARCHAR(255)` | `NOT NULL`                      | Hashed password.                             |
| `bio`               | `TEXT`         | `NULLABLE`                      | A short biography.                           |
| `profile_picture_url` | `VARCHAR(255)` | `NULLABLE`                      | URL to the user's profile image.             |
| `is_organization`   | `BOOLEAN`      | `NOT NULL`, `DEFAULT FALSE`     | Differentiates between users and organizations. |
| `created_at`        | `TIMESTAMP`    | `NOT NULL`, `DEFAULT NOW()`     | Timestamp of user creation.                  |

**Note on Password Hashing:** Never store passwords in plain text. Use a strong hashing algorithm like **bcrypt** or **Argon2** to securely store the password hash.

---

### `followers` Table
This is a junction table that models the many-to-many relationship between users. This table is the core of our social graph.

| Column Name    | Data Type   | Constraints                                   | Description                              |
| -------------- | ----------- | --------------------------------------------- | ---------------------------------------- |
| `follower_id`  | `UUID`      | `NOT NULL`, `FOREIGN KEY (users.user_id)`     | The user who is following.               |
| `following_id` | `UUID`      | `NOT NULL`, `FOREIGN KEY (users.user_id)`     | The user being followed.                 |
| `created_at`   | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()`                   | Timestamp of when the follow happened.   |

* **Primary Key:** The combination of (`follower_id`, `following_id`) is a **PRIMARY KEY** to prevent duplicate follow relationships.
* **Indexes:** Create a B-Tree index on both `follower_id` and `following_id` for quick lookups when retrieving a user's followers or who they are following.

---

## 2. Data Flow and Logic

### Registration
A new user's email and password are sent to the service. The service hashes the password, creates a new user record, and returns a success message with their `user_id`.

### Login
The service receives an email and password. It retrieves the user's record by email, hashes the provided password, and compares it to the stored hash. If they match, it generates a **JSON Web Token (JWT)** containing the `user_id` and returns it. This token will be used by the API Gateway for all subsequent authenticated requests.

### Following
A request to `POST /users/alex/follow` with a target ID (`{"target_user_id": "uuid-456"}`) is received. The service inserts a new row into the `followers` table with `follower_id` = alex_id and `following_id` = uuid-456. It then publishes an event to Kafka (e.g., `user_followed`) to be consumed by the Notification Service.

### Counting Followers
The `followers_count` and `following_count` in the `GET /users/{user_id}` response are not stored in the `users` table to avoid data redundancy. Instead, they are calculated on-the-fly using a `COUNT()` SQL query on the `followers` table. This ensures the numbers are always accurate.

**To get a user's follower count:**
```
SELECT COUNT(*) FROM followers WHERE following_id = 'user_id';
```