# Discovery & Recommendation Service

## Overview

The Discovery & Recommendation Service enhances user engagement by providing personalized event and organization recommendations. It operates independently, consuming data from Kafka and serving pre-calculated recommendations via a fast, lightweight API.

### Core Features
- **Personalized Recommendations**: Suggests events and organizations based on user behavior.
- **Trending Events**: Highlights popular events based on RSVP and view counts.
- **Scalable Architecture**: Decoupled from core services, ensuring high availability and performance.

### Technology Stack
| **Component**         | **Technology**          | **Role**                                                                 |
|------------------------|-------------------------|---------------------------------------------------------------------------|
| **Data Ingestion**     | Apache Kafka            | Streams event and user interaction data.                                 |
| **Data Storage**       | MongoDB                | Stores raw event features and user profiles.                            |
| **Recommendation Cache** | Redis                 | Caches pre-calculated recommendations for fast retrieval.               |
| **Language/Framework** | Python (Flask)         | Implements recommendation logic and API endpoints.                      |

### Architecture
1. **Data Ingestion**: Consumes data from Kafka topics (`events`, `user_interactions`).
2. **Processing**: Updates MongoDB with event features and user profiles.
3. **Model Updates**: Periodically calculates recommendations and stores them in Redis.
4. **API Serving**: Provides fast access to recommendations via RESTful endpoints.