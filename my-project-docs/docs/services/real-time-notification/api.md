# Real-time Notification Service API

The Real-time Notification Service provides minimal REST API endpoints alongside WebSocket connections for real-time notifications. The service primarily communicates through WebSockets and internal Kafka consumption for real-time delivery, with REST endpoints for offline notification management.

## Base URL

```
https://api.yourplatform.com/v1
```

## WebSocket URL

```
wss://notifications.yourplatform.com/v1/ws
```

## Authentication

All endpoints require authentication via JWT token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

For WebSocket connections, authentication is handled during the connection handshake.

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

## REST API Endpoints

### 1. Get Offline Notifications

Retrieves stored notifications for a user when they first log in, before the WebSocket connection is established. This endpoint fetches any notifications missed while the user was offline.

**Endpoint:** `GET /notifications/{user_id}`

**Authentication:** Required

**Query Parameters:**
- `limit` (integer): Maximum number of notifications to return (default: 50, max: 200)
- `offset` (integer): Number of notifications to skip for pagination (default: 0)
- `unread_only` (boolean): Return only unread notifications (default: false)
- `since` (string): ISO 8601 timestamp to get notifications after this time
- `types` (string): Comma-separated list of notification types to filter

#### Response

**Success (200 OK):**
```json
{
  "notifications": [
    {
      "id": "notif-uuid-123",
      "user_id": "uuid-456",
      "type": "follow",
      "title": "New Follower",
      "content": "alex_codes started following you",
      "timestamp": "2025-01-16T14:30:00Z",
      "read": false,
      "data": {
        "follower_id": "uuid-alex",
        "follower_username": "alex_codes",
        "follower_avatar": "https://cdn.yourplatform.com/profiles/uuid-alex.jpg"
      },
      "action_url": "/profile/alex_codes",
      "expires_at": "2025-02-16T14:30:00Z"
    },
    {
      "id": "notif-uuid-124",
      "user_id": "uuid-456",
      "type": "event_reminder",
      "title": "Event Reminder",
      "content": "AI Workshop starts in 1 hour",
      "timestamp": "2025-01-16T13:00:00Z",
      "read": false,
      "data": {
        "event_id": "event-uuid-789",
        "event_title": "AI Workshop",
        "event_time": "2025-01-16T14:00:00Z"
      },
      "action_url": "/events/event-uuid-789",
      "priority": "high"
    },
    {
      "id": "notif-uuid-125",
      "user_id": "uuid-456",
      "type": "post_like",
      "title": "Post Liked",
      "content": "sarah_dev liked your post about machine learning",
      "timestamp": "2025-01-16T12:15:00Z",
      "read": true,
      "data": {
        "post_id": "post-uuid-101",
        "liker_id": "uuid-sarah",
        "liker_username": "sarah_dev",
        "post_excerpt": "Today I learned about neural networks..."
      },
      "action_url": "/posts/post-uuid-101"
    }
  ],
  "pagination": {
    "total": 25,
    "limit": 50,
    "offset": 0,
    "has_more": false
  },
  "unread_count": 12,
  "last_updated": "2025-01-16T14:30:00Z"
}
```

**Error (401 Unauthorized):**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required",
    "details": {}
  }
}
```

**Error (403 Forbidden):**
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Cannot access notifications for another user",
    "details": {}
  }
}
```

---

### 2. Mark Notification as Read

Updates the status of a stored notification in the database to mark it as read.

**Endpoint:** `POST /notifications/{notification_id}/read`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "status": "Marked as read",
  "notification_id": "notif-uuid-123",
  "marked_at": "2025-01-16T15:00:00Z",
  "user_id": "uuid-456"
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "NOTIFICATION_NOT_FOUND",
    "message": "Notification not found",
    "details": {}
  }
}
```

**Error (403 Forbidden):**
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Cannot mark another user's notification as read",
    "details": {}
  }
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "ALREADY_READ",
    "message": "Notification is already marked as read",
    "details": {}
  }
}
```


---

## Additional REST Endpoints

### 1. Mark All Notifications as Read

Marks all notifications for a user as read.

**Endpoint:** `POST /notifications/{user_id}/read-all`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "status": "All notifications marked as read",
  "user_id": "uuid-456",
  "count": 12,
  "marked_at": "2025-01-16T15:30:00Z"
}
```

---

### 2. Delete Notification

Removes a notification from the user's notification list.

**Endpoint:** `DELETE /notifications/{notification_id}`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "status": "Notification deleted",
  "notification_id": "notif-uuid-123",
  "deleted_at": "2025-01-16T16:00:00Z"
}
```

---

### 3. Get Notification Settings

Retrieves user's notification preferences.

**Endpoint:** `GET /notifications/{user_id}/settings`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "user_id": "uuid-456",
  "settings": {
    "email_notifications": true,
    "push_notifications": true,
    "in_app_notifications": true,
    "notification_types": {
      "follow": {
        "enabled": true,
        "email": false,
        "push": true,
        "in_app": true
      },
      "post_like": {
        "enabled": true,
        "email": false,
        "push": false,
        "in_app": true
      },
      "event_reminder": {
        "enabled": true,
        "email": true,
        "push": true,
        "in_app": true
      },
      "comment": {
        "enabled": true,
        "email": true,
        "push": true,
        "in_app": true
      }
    },
    "quiet_hours": {
      "enabled": true,
      "start_time": "22:00",
      "end_time": "08:00",
      "timezone": "America/New_York"
    }
  },
  "updated_at": "2025-01-15T10:00:00Z"
}
```


---

### 4. Update Notification Settings

Updates user's notification preferences.

**Endpoint:** `PUT /notifications/{user_id}/settings`

**Authentication:** Required

#### Request

```json
{
  "email_notifications": true,
  "push_notifications": false,
  "notification_types": {
    "follow": {
      "enabled": true,
      "email": false,
      "push": false,
      "in_app": true
    },
    "event_reminder": {
      "enabled": true,
      "email": true,
      "push": true,
      "in_app": true
    }
  },
  "quiet_hours": {
    "enabled": true,
    "start_time": "23:00",
    "end_time": "07:00",
    "timezone": "America/New_York"
  }
}
```

#### Response

**Success (200 OK):**
```json
{
  "status": "Notification settings updated",
  "user_id": "uuid-456",
  "updated_at": "2025-01-16T17:00:00Z"
}
```


---

## WebSocket API

### Connection

To establish a WebSocket connection for real-time notifications:

```javascript
const ws = new WebSocket('wss://notifications.yourplatform.com/v1/ws?token=<jwt_token>');
```

### Authentication

Send authentication message immediately after connection:

```javascript
ws.send(JSON.stringify({
  type: 'auth',
  token: '<jwt_token>'
}));
```

### Message Types

#### Incoming Messages (Server to Client)

**Authentication Success:**
```json
{
  "type": "auth_success",
  "user_id": "uuid-456",
  "connected_at": "2025-01-16T18:00:00Z"
}
```

**New Notification:**
```json
{
  "type": "notification",
  "data": {
    "id": "notif-uuid-126",
    "user_id": "uuid-456",
    "type": "follow",
    "title": "New Follower",
    "content": "john_doe started following you",
    "timestamp": "2025-01-16T18:05:00Z",
    "read": false,
    "data": {
      "follower_id": "uuid-john",
      "follower_username": "john_doe"
    },
    "action_url": "/profile/john_doe"
  }
}
```

**Notification Updated:**
```json
{
  "type": "notification_updated",
  "data": {
    "id": "notif-uuid-123",
    "read": true,
    "updated_at": "2025-01-16T18:10:00Z"
  }
}
```

**Connection Keep-Alive:**
```json
{
  "type": "ping",
  "timestamp": "2025-01-16T18:15:00Z"
}
```

#### Outgoing Messages (Client to Server)

**Mark as Read:**
```json
{
  "type": "mark_read",
  "notification_id": "notif-uuid-123"
}
```

**Pong Response:**
```json
{
  "type": "pong",
  "timestamp": "2025-01-16T18:15:00Z"
}
```

**Subscribe to Types:**
```json
{
  "type": "subscribe",
  "notification_types": ["follow", "event_reminder", "post_like"]
}
```
---

## Notification Types

The system supports various notification types:

| Type | Description | Example |
|------|-------------|---------|
| `follow` | User started following | "alex_codes started following you" |
| `unfollow` | User unfollowed (optional) | "alex_codes unfollowed you" |
| `post_like` | Post was liked | "sarah_dev liked your post" |
| `post_comment` | Comment on post | "john_doe commented on your post" |
| `event_reminder` | Event starting soon | "AI Workshop starts in 1 hour" |
| `event_update` | Event was updated | "AI Workshop location changed" |
| `event_cancelled` | Event was cancelled | "AI Workshop has been cancelled" |
| `mention` | User was mentioned | "alex_codes mentioned you in a post" |
| `system` | System announcements | "Platform maintenance scheduled" |

---

## Rate Limiting

REST endpoints are subject to rate limiting:

- **GET /notifications/{user_id}**: 120 requests per hour
- **POST /notifications/*/read**: 300 requests per hour
- **Settings endpoints**: 60 requests per hour

WebSocket connections are limited to:
- **Connection attempts**: 10 per minute
- **Message rate**: 100 messages per minute per connection

---

## Error Codes Reference

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOTIFICATION_NOT_FOUND` | 404 | Notification does not exist |
| `ALREADY_READ` | 400 | Notification already marked as read |
| `INVALID_INPUT` | 400 | Request validation failed |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `WEBSOCKET_AUTH_FAILED` | 4001 | WebSocket authentication failed |
| `WEBSOCKET_RATE_LIMITED` | 4029 | WebSocket rate limit exceeded |
| `INTERNAL_ERROR` | 500 | Internal server error |

---
