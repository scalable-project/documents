# Event Management Service (EMS) Overview

The Event Management Service (EMS) is the single, authoritative microservice responsible for the lifecycle and state of all campus events. It manages event creation, modification, retrieval, and deletion. In the event-driven architecture of Campus Connect, the EMS acts as a primary event producer, broadcasting every critical change in event state to the rest of the system via Apache Kafka.

---

## 1. Core Responsibilities

The EMS defines the "Bounded Context" for event data, ensuring data integrity and consistency for all event entities:

- **Event Lifecycle Management:** Handles all CRUD operations for the Event entity.
- **Data Integrity:** Validates all incoming event data (e.g., ensuring dates are in the future, locations are valid, and required fields are present).
- **Event Production:** Publishes critical state-change events (`event_created`, `event_updated`, etc.) to Kafka. This decouples the service from downstream consumers (like the Notification Service and Recommendation Service).
- **Attendee Count Consistency:** Listens for external RSVP events to maintain an accurate and up-to-date count of attendees for each event.

---

## 2. Technology Stack

| **Component**         | **Technology**            | **Rationale**                                                                 |
|------------------------|---------------------------|-------------------------------------------------------------------------------|
| **Language/Framework** | Not decided     | NA. |
| **Database**           | PostgreSQL               | Provides strong relational integrity and transactional guarantees for crucial event details. |
| **Communication**      | REST API (Synchronous), Apache Kafka (Asynchronous) | REST is used for immediate client requests (e.g., create event); Kafka is used for broadcasting the resulting state change. |

---

## 3. API Summary (REST Endpoints)

The EMS exposes standard RESTful endpoints for resource management. These are consumed primarily by the API Gateway.

| **Endpoint**          | **Method** | **Function**                                      | **Authentication Required** |
|------------------------|-----------|--------------------------------------------------|-----------------------------|
| `/events`             | POST      | Create a new event, published by a user/organization. | Yes (Authenticated `host_id`) |
| `/events/{event_id}`  | GET       | Retrieve the full details of a single event.      | No (Public access)          |
| `/events/{event_id}`  | PUT       | Update an event's details (requires matching `host_id` validation). | Yes (Host Authorization)    |
| `/events/{event_id}`  | DELETE    | Delete an event (requires matching `host_id` validation). | Yes (Host Authorization)    |
| `/events/search`      | GET       | Search and filter events (used for discovery).    | No (Public access)          |

---

## 4. Data Model (PostgreSQL)

The EMS owns the single `events` table.

| **Field Name**      | **Data Type**             | **Constraints**          | **Description**                                                   |
|---------------------|---------------------------|--------------------------|-------------------------------------------------------------------|
| `event_id`          | UUID                     | Primary Key              | Unique identifier.                                               |
| `host_id`           | UUID                     | Not Null                 | ID of the hosting user/organization (links conceptually to the USS). |
| `title`             | VARCHAR                  | Not Null                 | Event name.                                                      |
| `date_time`         | TIMESTAMP WITH TIME ZONE | Not Null                 | Scheduled start time.                                            |
| `location`          | VARCHAR                  | Not Null                 | Venue or virtual link.                                           |
| `attendees_count`   | INTEGER                  | Default 0                | Maintained asynchronously by consuming Kafka RSVP events.        |

---

## 5. Event Production and Consumption

The EMS acts as a key producer and consumer in the system, demonstrating the effective use of event-driven patterns:

| **Role**   | **Kafka Topic**       | **Event Type(s)**           | **Trigger/Action**                                              |
|------------|-----------------------|-----------------------------|-----------------------------------------------------------------|
| Producer   | `events`              | `event_created`, `event_updated`, `event_deleted` | Triggered internally after any successful write operation to the EMS's PostgreSQL DB. |
| Consumer   | `event_interactions`  | `user_rsvp_added`           | Triggered by an external RSVP action. The EMS consumes this event and updates its local `attendees_count` to maintain data consistency. |