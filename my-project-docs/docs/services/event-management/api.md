# Event Management Service API

The Event Management Service handles all event-related operations including creating, updating, deleting, and searching for events on the platform.

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

## Event Management

### 1. Create Event

Creates a new event on the platform.

**Endpoint:** `POST /events`

**Authentication:** Required

#### Request

```json
{
  "title": "AI Workshop",
  "description": "Learn the fundamentals of artificial intelligence and machine learning. This hands-on workshop will cover neural networks, deep learning, and practical AI applications.",
  "date_time": "2025-10-27T14:00:00Z",
  "location": "Room 101, Computer Science Building",
  "host_id": "uuid-123",
  "capacity": 50,
  "is_public": true,
  "tags": ["AI", "Machine Learning", "Workshop", "Technology"],
  "registration_required": true,
  "registration_deadline": "2025-10-25T23:59:59Z"
}
```

#### Response

**Success (201 Created):**
```json
{
  "event_id": "uuid-456",
  "title": "AI Workshop",
  "message": "Event created successfully",
  "created_at": "2025-01-16T10:30:00Z",
  "event_url": "https://yourplatform.com/events/uuid-456"
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Validation failed",
    "details": {
      "date_time": "Event date must be in the future",
      "location": "Location is required"
    }
  }
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


---

### 2. Get Event Details

Retrieves detailed information about a specific event.

**Endpoint:** `GET /events/{event_id}`

**Authentication:** Optional (public events accessible without auth)

#### Response

**Success (200 OK):**
```json
{
  "event_id": "uuid-456",
  "title": "AI Workshop",
  "description": "Learn the fundamentals of artificial intelligence and machine learning. This hands-on workshop will cover neural networks, deep learning, and practical AI applications.",
  "date_time": "2025-10-27T14:00:00Z",
  "location": "Room 101, Computer Science Building",
  "host_id": "uuid-123",
  "host": {
    "user_id": "uuid-123",
    "username": "professor_alex",
    "full_name": "Prof. Alex Johnson",
    "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-123.jpg"
  },
  "attendees_count": 50,
  "capacity": 100,
  "is_public": true,
  "tags": ["AI", "Machine Learning", "Workshop", "Technology"],
  "registration_required": true,
  "registration_deadline": "2025-10-25T23:59:59Z",
  "status": "upcoming",
  "created_at": "2025-01-16T10:30:00Z",
  "updated_at": "2025-01-16T12:15:00Z",
  "is_attending": false,
  "can_edit": false,
  "images": [
    "https://cdn.yourplatform.com/events/uuid-456/cover.jpg"
  ]
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "EVENT_NOT_FOUND",
    "message": "Event not found",
    "details": {}
  }
}
```



---

### 3. Update Event

Updates an existing event. Only the event host can update their events.

**Endpoint:** `PUT /events/{event_id}`

**Authentication:** Required (must be event host)

#### Request

```json
{
  "title": "AI Workshop - Location Changed",
  "location": "Room 202, Computer Science Building",
  "description": "Updated description with new location details.",
  "capacity": 75
}
```

#### Response

**Success (200 OK):**
```json
{
  "event_id": "uuid-456",
  "title": "AI Workshop - Location Changed",
  "message": "Event updated successfully",
  "updated_at": "2025-01-16T15:30:00Z",
  "changes": [
    "title",
    "location",
    "description",
    "capacity"
  ]
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "INVALID_UPDATE",
    "message": "Cannot update past events",
    "details": {}
  }
}
```

**Error (401 Unauthorized):**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Only event host can update this event",
    "details": {}
  }
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "EVENT_NOT_FOUND",
    "message": "Event not found",
    "details": {}
  }
}
```


---

### 4. Delete Event

Deletes an event. Only the event host can delete their events.

**Endpoint:** `DELETE /events/{event_id}`

**Authentication:** Required (must be event host)

#### Response

**Success (200 OK):**
```json
{
  "message": "Event deleted successfully",
  "event_id": "uuid-456",
  "deleted_at": "2025-01-16T16:00:00Z"
}
```

**Error (401 Unauthorized):**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Only event host can delete this event",
    "details": {}
  }
}
```

**Error (404 Not Found):**
```json
{
  "error": {
    "code": "EVENT_NOT_FOUND",
    "message": "Event not found",
    "details": {}
  }
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "CANNOT_DELETE_STARTED_EVENT",
    "message": "Cannot delete events that have already started",
    "details": {}
  }
}
```


---

### 5. Search Events

Search for events based on various criteria.

**Endpoint:** `GET /events/search`

**Authentication:** Optional

**Query Parameters:**
- `query` (string): Search term for event title and description
- `date` (string): Filter by specific date (YYYY-MM-DD format)
- `date_from` (string): Filter events starting from this date
- `date_to` (string): Filter events ending before this date
- `location` (string): Filter by location
- `host_id` (string): Filter by event host
- `tags` (string): Comma-separated list of tags
- `is_public` (boolean): Filter by public/private events
- `status` (string): Filter by event status (`upcoming`, `ongoing`, `completed`, `cancelled`)
- `page` (integer): Page number for pagination (default: 1)
- `limit` (integer): Number of results per page (default: 20, max: 100)
- `sort_by` (string): Sort results by (`date`, `created_at`, `title`, `attendees_count`)
- `sort_order` (string): Sort order (`asc`, `desc`)

#### Response

**Success (200 OK):**
```json
{
  "events": [
    {
      "event_id": "uuid-456",
      "title": "AI Workshop",
      "description": "Learn the fundamentals of artificial intelligence...",
      "date_time": "2025-10-27T14:00:00Z",
      "location": "Room 101, Computer Science Building",
      "host": {
        "user_id": "uuid-123",
        "username": "professor_alex",
        "full_name": "Prof. Alex Johnson"
      },
      "attendees_count": 50,
      "capacity": 100,
      "tags": ["AI", "Machine Learning", "Workshop"],
      "status": "upcoming",
      "is_public": true,
      "cover_image": "https://cdn.yourplatform.com/events/uuid-456/cover.jpg"
    },
    {
      "event_id": "uuid-789",
      "title": "Machine Learning Seminar",
      "description": "Advanced topics in machine learning...",
      "date_time": "2025-10-28T16:00:00Z",
      "location": "Auditorium A, Main Campus",
      "host": {
        "user_id": "uuid-321",
        "username": "dr_smith",
        "full_name": "Dr. Jane Smith"
      },
      "attendees_count": 120,
      "capacity": 200,
      "tags": ["AI", "Machine Learning", "Seminar"],
      "status": "upcoming",
      "is_public": true,
      "cover_image": "https://cdn.yourplatform.com/events/uuid-789/cover.jpg"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 2,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  },
  "filters_applied": {
    "query": "AI",
    "date": "2025-10-27"
  }
}
```

---

### 6. Get Events by Host

Retrieve all events created by a specific host.

**Endpoint:** `GET /events/host/{host_id}`

**Authentication:** Optional

**Query Parameters:**
- `status` (string): Filter by event status
- `page` (integer): Page number for pagination
- `limit` (integer): Number of results per page

#### Response

**Success (200 OK):**
```json
{
  "host": {
    "user_id": "uuid-123",
    "username": "professor_alex",
    "full_name": "Prof. Alex Johnson"
  },
  "events": [
    {
      "event_id": "uuid-456",
      "title": "AI Workshop",
      "date_time": "2025-10-27T14:00:00Z",
      "location": "Room 101, Computer Science Building",
      "attendees_count": 50,
      "status": "upcoming"
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

### 7. Get Upcoming Events

Retrieve a list of upcoming events.

**Endpoint:** `GET /events/upcoming`

**Authentication:** Optional

**Query Parameters:**
- `limit` (integer): Number of events to return (default: 10, max: 50)
- `location` (string): Filter by location
- `tags` (string): Filter by tags

#### Response

**Success (200 OK):**
```json
{
  "events": [
    {
      "event_id": "uuid-456",
      "title": "AI Workshop",
      "date_time": "2025-10-27T14:00:00Z",
      "location": "Room 101, Computer Science Building",
      "host": {
        "user_id": "uuid-123",
        "username": "professor_alex",
        "full_name": "Prof. Alex Johnson"
      },
      "attendees_count": 50,
      "tags": ["AI", "Workshop"],
      "time_until_event": "2 days"
    }
  ]
}
```



---

### 8.Get Event Categories

Retrieve available event categories/tags.

**Endpoint:** `GET /events/categories`

**Authentication:** Optional

#### Response

**Success (200 OK):**
```json
{
  "categories": [
    {
      "name": "Technology",
      "count": 145,
      "subcategories": ["AI", "Machine Learning", "Web Development", "Mobile Development"]
    },
    {
      "name": "Business",
      "count": 89,
      "subcategories": ["Entrepreneurship", "Marketing", "Finance", "Leadership"]
    },
    {
      "name": "Academic",
      "count": 234,
      "subcategories": ["Research", "Seminar", "Conference", "Workshop"]
    }
  ]
}
```

---

## 9. Event Registration

### Register for Event

Register the authenticated user for an event.

**Endpoint:** `POST /events/{event_id}/register`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "message": "Successfully registered for event",
  "event_id": "uuid-456",
  "registration_id": "uuid-reg-123",
  "registered_at": "2025-01-16T18:00:00Z"
}
```

**Error (400 Bad Request):**
```json
{
  "error": {
    "code": "REGISTRATION_CLOSED",
    "message": "Registration deadline has passed",
    "details": {}
  }
}
```


---

### 10. Unregister from Event

Remove registration for an event.

**Endpoint:** `DELETE /events/{event_id}/register`

**Authentication:** Required

#### Response

**Success (200 OK):**
```json
{
  "message": "Successfully unregistered from event",
  "event_id": "uuid-456",
  "unregistered_at": "2025-01-16T19:00:00Z"
}
```


---

### 11. Get Event Attendees

Retrieve the list of users registered for an event.

**Endpoint:** `GET /events/{event_id}/attendees`

**Authentication:** Required (event host or registered attendee)

**Query Parameters:**
- `page` (integer): Page number for pagination
- `limit` (integer): Number of results per page

#### Response

**Success (200 OK):**
```json
{
  "attendees": [
    {
      "user_id": "uuid-attendee-1",
      "username": "student_bob",
      "full_name": "Bob Wilson",
      "profile_picture_url": "https://cdn.yourplatform.com/profiles/uuid-attendee-1.jpg",
      "registered_at": "2025-01-15T14:30:00Z"
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

## Rate Limiting

All endpoints are subject to rate limiting:

- **Create/Update/Delete operations**: 30 requests per hour
- **Read operations**: 300 requests per hour
- **Search operations**: 120 requests per hour

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 295
X-RateLimit-Reset: 1642345200
```

---

## Webhooks

The Event Management Service can send webhooks for the following events:

### Event Events
- `event.created` - When a new event is created
- `event.updated` - When an event is updated
- `event.deleted` - When an event is deleted
- `event.started` - When an event starts (based on date_time)
- `event.ended` - When an event ends

### Registration Events
- `event.user_registered` - When a user registers for an event
- `event.user_unregistered` - When a user unregisters from an event

### Webhook Payload Example

```json
{
  "event": "event.created",
  "timestamp": "2025-01-16T10:30:00Z",
  "data": {
    "event_id": "uuid-456",
    "title": "AI Workshop",
    "host_id": "uuid-123",
    "date_time": "2025-10-27T14:00:00Z",
    "location": "Room 101, Computer Science Building"
  }
}
```

---

## Error Codes Reference

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_INPUT` | 400 | Request validation failed |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `EVENT_NOT_FOUND` | 404 | Event does not exist |
| `CANNOT_DELETE_STARTED_EVENT` | 400 | Cannot delete events that have started |
| `REGISTRATION_CLOSED` | 400 | Registration deadline has passed |
| `EVENT_FULL` | 400 | Event has reached capacity |
| `ALREADY_REGISTERED` | 409 | User already registered for event |
| `NOT_REGISTERED` | 400 | User not registered for event |
| `INVALID_UPDATE` | 400 | Cannot update past events |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Internal server error |

---
