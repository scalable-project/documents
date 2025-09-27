# User & Social Service API

The User & Social Service is the foundation service that handles user management, authentication, and social connections (following/followers) for the platform.

## Base URL

```
https://api.yourplatform.com/v1
```

## Authentication

Most endpoints require authentication via JWT token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

## Error Response Format

All error responses follow this format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {}
  }
}
```

---

## User Management

### 1. Register User

Creates a new user account.

**Endpoint:** `POST /users/register`

**Authentication:** None required

#### Request

```json
{
  "username": "alex_codes",
  "email": "alex@university.edu",
  "password": "securePassword123",
  "full_name": "Alex Johnson",
  "bio": "Computer Science student passionate about AI"
}
```

#### Response

**Success (201 Created):**
```json
{
  "user_id": "uuid-123",
  "username": "alex_codes",
  "email": "alex@university.edu",
  "full_name": "Alex Johnson",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Error (409 Conflict):**
```json
{
  "error": {
    "code": "USER_EXISTS",
    "message": "Username or email already exists",
    "details": {
      "field": "email"
    }
  }
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Validation failed",
    "details": {
      "password": "Password must be at least 8 characters long"
    }
  }
}
```


---

### 2. Login User

Authenticates a user and returns a JWT token.

**Endpoint:** `POST /users/login`

**Authentication:** None required

#### Request

```json
{
  "email": "alex@university.edu",
  "password": "securePassword123"
}
```

#### Response

**Success (200 OK):**
```json
{
  "user_id": "uuid-123",
  "username": "alex_codes",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_at": "2024-01-16T10:30:00Z"
}
```

**Error (401 Unauthorized):**
```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password",
    "details": {}
  }
}
```


---

### 3. Get User Profile

Retrieves a user's public profile information.

**Endpoint:** `GET /users/{user_id}`

**Authentication:** Optional (returns more details if authenticated)

#### Response

**Success (200 OK):**
```json
{
  "user_id": "uuid-123",
  "username": "alex_codes",
  "full_name": "Alex Johnson",
  "bio": "Computer Science student passionate about AI",
  "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-123.jpg",
  "followers_count": 50,
  "following_count": 120,
  "posts_count": 25,
  "joined_at": "2024-01-15T10:30:00Z",
  "is_verified": false,
  "is_following": false
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found",
    "details": {}
  }
}
```

---

### 4. Update User Profile

Updates the authenticated user's profile information.

**Endpoint:** `PUT /users/{user_id}`

**Authentication:** Required (user can only update their own profile)

#### Request

```json
{
  "full_name": "Alexander Johnson",
  "bio": "CS student & AI researcher",
  "profile_picture_url": "https://cdn.yourplatform.com/profiles/new-pic.jpg"
}
```

#### Response

**Success (200 OK):**
```json
{
  "user_id": "uuid-123",
  "username": "alex_codes",
  "full_name": "Alexander Johnson",
  "bio": "CS student & AI researcher",
  "profile_picture_url": "https://cdn.yourplatform.com/profiles/new-pic.jpg",
  "updated_at": "2024-01-16T14:25:00Z"
}
```

---

## Social Connections

### 1. Follow User

Creates a following relationship between the authenticated user and a target user.

**Endpoint:** `POST /users/{user_id}/follow`

**Authentication:** Required

#### Request

```json
{
  "target_user_id": "uuid-456"
}
```

#### Response

**Success (200 OK):**
```json
{
  "status": "success",
  "message": "Following user uuid-456",
  "following": {
    "user_id": "uuid-456",
    "username": "campus_robotics",
    "followed_at": "2024-01-16T15:30:00Z"
  }
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "Target user not found",
    "details": {}
  }
}
```

**Error (409 Conflict):**
```json
{
  "error": {
    "code": "ALREADY_FOLLOWING",
    "message": "You are already following this user",
    "details": {}
  }
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "CANNOT_FOLLOW_SELF",
    "message": "You cannot follow yourself",
    "details": {}
  }
}
```

---

### 2. Unfollow User

Removes a following relationship between the authenticated user and a target user.

**Endpoint:** `DELETE /users/{user_id}/follow/{target_user_id}`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "status": "success",
  "message": "Successfully unfollowed user uuid-456",
  "unfollowed": {
    "user_id": "uuid-456",
    "username": "campus_robotics",
    "unfollowed_at": "2024-01-16T16:00:00Z"
  }
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "FOLLOW_RELATIONSHIP_NOT_FOUND",
    "message": "Follow relationship does not exist",
    "details": {}
  }
}
```

---

### 3. Get Following List

Retrieves the list of users and organizations that a user is following.

**Endpoint:** `GET /users/{user_id}/following`

**Authentication:** Optional

**Query Parameters:**
- `page` (integer): Page number for pagination (default: 1)
- `limit` (integer): Number of results per page (default: 20, max: 100)
- `search` (string): Search query to filter following list

#### Response

**Success (200 OK):**
```json
{
  "following": [
    {
      "user_id": "uuid-456",
      "username": "campus_robotics",
      "full_name": "Campus Robotics Club",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-456.jpg",
      "is_verified": true,
      "followed_at": "2024-01-10T09:15:00Z"
    },
    {
      "user_id": "uuid-789",
      "username": "professor_jane",
      "full_name": "Prof. Jane Smith",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-789.jpg",
      "is_verified": true,
      "followed_at": "2024-01-08T14:20:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 120,
    "total_pages": 6,
    "has_next": true,
    "has_prev": false
  }
}
```


---

### Get Followers List

Retrieves the list of users following a specific user or organization.

**Endpoint:** `GET /users/{user_id}/followers`

**Authentication:** Optional

**Query Parameters:**
- `page` (integer): Page number for pagination (default: 1)
- `limit` (integer): Number of results per page (default: 20, max: 100)
- `search` (string): Search query to filter followers list

#### Response

**Success (200 OK):**
```json
{
  "followers": [
    {
      "user_id": "uuid-101",
      "username": "ben_student",
      "full_name": "Ben Martinez",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-101.jpg",
      "is_verified": false,
      "followed_at": "2024-01-12T11:30:00Z"
    },
    {
      "user_id": "uuid-112",
      "username": "chris_tech",
      "full_name": "Chris Thompson",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-112.jpg",
      "is_verified": false,
      "followed_at": "2024-01-11T16:45:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 50,
    "total_pages": 3,
    "has_next": true,
    "has_prev": false
  }
}
```


---

### Search Users

Search for users by username, full name, or bio.

**Endpoint:** `GET /users/search`

**Authentication:** Optional

**Query Parameters:**
- `q` (string, required): Search query
- `page` (integer): Page number for pagination (default: 1)
- `limit` (integer): Number of results per page (default: 20, max: 100)
- `type` (string): Filter by user type (`user`, `organization`, `all`) (default: `all`)

#### Response

**Success (200 OK):**
```json
{
  "users": [
    {
      "user_id": "uuid-456",
      "username": "alex_codes",
      "full_name": "Alex Johnson",
      "bio": "Computer Science student passionate about AI",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-456.jpg",
      "followers_count": 50,
      "is_verified": false,
      "is_following": false
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  }
}
```

---

### Get Follow Recommendations

Get recommended users to follow based on the authenticated user's interests and connections.

**Endpoint:** `GET /users/{user_id}/recommendations`

**Authentication:** Required

**Query Parameters:**
- `limit` (integer): Number of recommendations (default: 10, max: 50)

#### Response

**Success (200 OK):**
```json
{
  "recommendations": [
    {
      "user_id": "uuid-999",
      "username": "ai_researcher",
      "full_name": "Dr. Sarah Chen",
      "bio": "AI Research Professor at Stanford",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-999.jpg",
      "followers_count": 5420,
      "mutual_connections": 3,
      "reason": "Followed by users with similar interests",
      "is_verified": true
    }
  ]
}
```


---

## User Statistics

### Get User Stats

Retrieve detailed statistics for a user's account.

**Endpoint:** `GET /users/{user_id}/stats`

**Authentication:** Required (user can only view their own detailed stats)

#### Response

**Success (200 OK):**
```json
{
  "user_id": "uuid-123",
  "stats": {
    "followers_count": 50,
    "following_count": 120,
    "posts_count": 25,
    "likes_received": 340,
    "comments_received": 89,
    "profile_views": 1250,
    "engagement_rate": 0.085,
    "growth": {
      "followers_this_week": 5,
      "followers_this_month": 18
    }
  },
  "generated_at": "2024-01-16T20:00:00Z"
}
```


---

## Rate Limiting

All endpoints are subject to rate limiting:

- **Authentication endpoints** (`/login`, `/register`): 5 requests per minute
- **Profile endpoints**: 100 requests per hour
- **Social endpoints**: 200 requests per hour
- **Search endpoints**: 60 requests per hour

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642345200
```

---

## Webhooks

The User Service can send webhooks for the following events:

### User Events
- `user.registered` - When a new user registers
- `user.profile_updated` - When a user updates their profile
- `user.verified` - When a user gets verified

### Social Events
- `user.followed` - When someone follows a user
- `user.unfollowed` - When someone unfollows a user

### Webhook Payload Example

```json
{
  "event": "user.followed",
  "timestamp": "2024-01-16T15:30:00Z",
  "data": {
    "follower_id": "uuid-123",
    "following_id": "uuid-456",
    "follower": {
      "user_id": "uuid-123",
      "username": "alex_codes"
    },
    "following": {
      "user_id": "uuid-456",
      "username": "campus_robotics"
    }
  }
}
```

---

## Error Codes Reference

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `USER_EXISTS` | 409 | Username or email already exists |
| `INVALID_INPUT` | 400 | Request validation failed |
| `INVALID_CREDENTIALS` | 401 | Invalid login credentials |
| `USER_NOT_FOUND` | 404 | User does not exist |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `ALREADY_FOLLOWING` | 409 | Already following the user |
| `CANNOT_FOLLOW_SELF` | 400 | Cannot follow yourself |
| `FOLLOW_RELATIONSHIP_NOT_FOUND` | 404 | Follow relationship does not exist |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Internal server error |

---

