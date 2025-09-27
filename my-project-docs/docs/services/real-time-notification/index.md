# Overview of the Real-time Notification Service

The Real-time Notification Service acts as the central communication hub for the application. Its primary responsibility is to deliver instant, event-driven notifications to online users via WebSockets, ensuring that users are immediately informed of relevant activity, such as a new event being created by an organization they follow.

---
## Communication and Technology Stack

| **Component**          | **Technology**                                   | **Role in Service**                                                                                     |
|-------------------------|-------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **Event Ingestion**     | Apache Kafka (Consumer)                         | Subscribes to topics (`events`, `user_followed`, etc.) for continuous event processing.                 |
| **Real-time Push**      | WebSockets (e.g., using a library like Socket.IO or dedicated framework support in Go/Java) | Maintains persistent, two-way connections with every active user to push notifications instantly.       |
| **Caching/State**       | Redis                                           | Used to store the active mapping between `user_id` and their active WebSocket connection ID/Server ID for fast lookups. |
| **Notification Storage**| PostgreSQL/MongoDB (or a dedicated table in the Redis cluster) | Used to persist notifications for offline users, acting as a historical log.                           |

---

## The Notification Pipeline 📡

The service operates on a clear, step-by-step data flow when an event occurs:

1. **Consume from Kafka:** The process begins when the service consumes a message (e.g., `event_created`) from a central Kafka topic.
2. **Identify the Audience:** It extracts key information from the event, like the `host_id`, and makes a synchronous API call to the User & Social Service to fetch a list of all relevant users (e.g., the host's followers).
3. **Check Online Status:** For each user in the audience, the service performs a high-speed lookup in its Redis cache to determine if they are currently connected with an active WebSocket.
4. **Push or Store:**
   - **If Online:** A notification is pushed instantly to the user's device through their open WebSocket connection.
   - **If Offline:** The notification is stored in a persistent database (like PostgreSQL or MongoDB) to be delivered the next time the user logs in.

---
## Managing Connections at Scale ⚡

When a user connects, a record is created in Redis mapping their `user_id` to the specific server instance (`host_01`, `host_02`, etc.) and `connection_id` handling their session. This allows any service instance that consumes a Kafka event to know exactly where to send the final notification.

### Active User List (Redis Set)

- **Key:** `ws:active_users`
- **Value:** A simple set that maintains a list of all currently connected users, which can be used for quick health checks and metrics.

---

## Resilience and Reliability 🛡️

To prevent cascading failures, the service is built with resilience in mind:

- **Circuit Breaker Pattern:** When calling the User & Social Service, the Notification Service employs a Circuit Breaker pattern. If the User Service is slow or unavailable, the circuit breaker will "open," immediately failing the request and preventing the Notification Service from becoming blocked.
- **Graceful Failure Handling:** In the event of a failure, the service logs the notification for a retry at a later time, ensuring that no notifications are lost and the system remains operational.