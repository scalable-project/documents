## Real-time Service: Redis Cache Schema

The primary use of Redis is to act as a central registry for active WebSocket connections, which is essential for horizontal scaling. It allows any service instance to quickly locate which specific server is handling a user's connection.

| **Key Type** | **Key Format**          | **Value Type**                                      | **Description**                                                                 |
|--------------|--------------------------|----------------------------------------------------|---------------------------------------------------------------------------------|
| **Hash**     | `ws:user:{user_id}`      | `{ "server_instance": "...", "connection_id": "..." }` | Maps a user ID to the specific server instance and connection ID currently holding their WebSocket. |
| **Set**      | `ws:active_users`        | Set of `{user_id}`                                 | A list of all currently connected user IDs, which can be used for quick health checks or metrics. |

---

This two-key approach is highly efficient:
- The **Hash** provides O(1) lookup to find a specific user's connection details.
- The **Set** gives a simple way to track the overall active user base.