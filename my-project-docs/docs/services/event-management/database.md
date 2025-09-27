# Database Schema Data Flow
## Event Management Service: Database Schema

This service uses a single `events` table to store all event-related data in its PostgreSQL database.

| **Column Name**    | **Data Type**                | **Constraints**                              | **Description**                                                                 |
|---------------------|-----------------------------|----------------------------------------------|---------------------------------------------------------------------------------|
| `event_id`          | UUID                        | PRIMARY KEY, NOT NULL                        | Unique identifier for the event.                                               |
| `title`             | VARCHAR(255)               | NOT NULL                                    | The title of the event.                                                        |
| `description`       | TEXT                        | NOT NULL                                    | A detailed description of the event.                                           |
| `host_id`           | UUID                        | NOT NULL, FOREIGN KEY (`users.user_id`)      | The ID of the user or organization hosting the event. Links to the User Service. |
| `date_time`         | TIMESTAMP WITH TIME ZONE    | NOT NULL                                    | The scheduled start time and date of the event.                                |
| `location`          | VARCHAR(255)               | NOT NULL                                    | The physical location where the event will take place.                         |
| `attendees_count`   | INTEGER                    | NOT NULL, DEFAULT 0                         | The number of confirmed attendees.                                             |
| `created_at`        | TIMESTAMP WITH TIME ZONE    | NOT NULL, DEFAULT NOW()                     | Timestamp for when the event was created.                                      |
| `updated_at`        | TIMESTAMP WITH TIME ZONE    | NOT NULL, DEFAULT NOW()                     | Timestamp for when the event was last updated.                                 |


### Conceptual Foreign Key

The `host_id` links to a user ID in the User & Social Service. This demonstrates decentralized data management. The Event Service stores the ID but doesn't have a direct database-level foreign key constraint to the `users` table.


## 4. Data Flow and Logic

### Create Event Flow

1. The API Gateway receives a `POST /events` request. It authenticates the user and passes the request to this service.
2. The Event Management Service validates the input and inserts a new row into the `events` table.
3. After a successful database write, the service publishes an `event_created` message to a Kafka topic named `events`. This message contains key details like `event_id`, `host_id`, `title`, etc.
4. The service returns a `201 Created` response to the client.

---

### Update Event Flow

1. A `PUT /events/{event_id}` request is received. The service verifies that the authenticated user's ID matches the `host_id` of the event to ensure they have permission to update it.
2. The service updates the corresponding row in the `events` table.
3. It then publishes an `event_updated` message to the Kafka `events` topic. This is crucial for the Notification Service to alert attendees about changes.

---

### Data Consistency

- The `attendees_count` column is an interesting case. It will not be directly updated by this service.
- Instead, when an external service (like a future RSVP service) processes a user RSVP, it will publish an `rsvp_added` event to a Kafka topic.
- The Event Management Service will consume this event from Kafka and update its own `attendees_count` column.
- This ensures data consistency across services in a decoupled, asynchronous way.