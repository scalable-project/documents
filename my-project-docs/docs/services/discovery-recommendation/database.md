# Database Schema

## MongoDB Collections

### `event_features` Collection
Stores metadata and feature vectors for events.

| **Field Name**         | **Data Type**       | **Description**                                      |
|-------------------------|---------------------|-----------------------------------------------------|
| `event_id`             | UUID               | Unique identifier for the event.                   |
| `host_id`              | UUID               | ID of the hosting organization.                    |
| `description_keywords` | Array of Strings   | Keywords extracted from the event description.     |
| `category`             | String             | Event category (e.g., 'Tech', 'Sports').           |
| `feature_vector`       | Array of Floats    | Numerical vector for similarity calculations.      |

---

### `user_profiles` Collection
Stores user behavior and interest data.

| **Field Name**         | **Data Type**       | **Description**                                      |
|-------------------------|---------------------|-----------------------------------------------------|
| `user_id`              | UUID               | Unique identifier for the user.                    |
| `followed_orgs`        | Array of UUIDs     | List of organizations the user follows.            |
| `rsvp_history`         | Array of UUIDs     | List of events the user has RSVP'd to.             |
| `interest_scores`      | Object             | Map of categories to interest scores.              |

---

## Redis Cache Schema

### Key Structure
| **Key Format**                  | **Value Type**       | **Description**                                      |
|----------------------------------|---------------------|-----------------------------------------------------|
| `user:{user_id}:recommendations` | Array of UUIDs      | List of recommended event IDs for the user.         |
| `trending:events`               | Array of UUIDs      | List of currently trending event IDs.               |