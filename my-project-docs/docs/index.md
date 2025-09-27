# Campus Connect: Scalable University Event Platform

Campus Connect is a modern, event-driven platform designed to connect students and organizations by streamlining the creation, discovery, and real-time notification of campus events. Built on a microservices architecture, the project demonstrates best practices in scalability, resilience, and decentralized data management.

---

## 1. Executive Summary: Why Campus Connect?

Campus Connect solves the common campus problem of information fragmentation by replacing slow, siloed emails and bulletin boards with a single, real-time platform.

| **Principle**      | **Implementation**                                      | **Project Benefit**                                                                 |
|---------------------|---------------------------------------------------------|-------------------------------------------------------------------------------------|
| **Microservices**   | 4 independently deployable services (USS, EMS, RNS, DRS). | Enables independent scaling and technology choice.                                  |
| **Scalability**     | Asynchronous communication via Apache Kafka.            | Handles high traffic spikes (e.g., event announcements) without service failure.    |
| **Real-Time**       | WebSockets and the Real-time Notification Service (RNS). | Instantaneous push notifications for event updates and new posts.                  |
| **Resilience**      | Decoupled Architecture and Event Sourcing.              | A service failure (e.g., Recommendation Service outage) does not impact core functionality (event creation). |


---

## 2. Architecture Overview

Campus Connect is built on a loosely coupled, event-driven architecture (EDA) where microservices communicate primarily through a central message bus, Apache Kafka.

### Key Components & Communication

| **Component**               | **Primary Function**                                | **Data Store**                  | **Communication**                                                                                     |
|-----------------------------|----------------------------------------------------|---------------------------------|-------------------------------------------------------------------------------------------------------|
| **API Gateway**             | Single entry point, routing, and authentication.   | N/A                             | Synchronous REST to all services.                                                                    |
| **User & Social Service (USS)** | User registration, login, profile, following/followers. | PostgreSQL                      | REST API ↔ Gateway. Publishes to Kafka (`user_events`).                                               |
| **Event Management Service (EMS)** | Source of truth for all event details (CRUD).       | PostgreSQL                      | REST API ↔ Gateway. Publishes to Kafka (`events`). Consumes from Kafka (`event_interactions`).        |
| **Real-time Notification Service (RNS)** | Pushes alerts for new events and updates.          | Redis (Cache), DB (Offline)     | Consumes from Kafka (All Topics). Calls USS (REST). Pushes to clients (WebSockets).                  |
| **Discovery & Recommendation Service (DRS)** | Provides personalized event and organization suggestions. | MongoDB, Redis                  | Consumes from Kafka (All Topics). Serves clients (REST).                                              |


## 3. Core Features & User Journey

The system supports a full life cycle from user sign-up to real-time engagement.

---

### A. User Journey: Event Creation & Notification

| **Step**                | **Service Interaction**                          | **Technical Principle Demonstrated**                                                   |
|-------------------------|--------------------------------------------------|---------------------------------------------------------------------------------------|
| **1. Create Event**     | Client → API Gateway → EMS.                      | **Synchronous REST:** Fast, guaranteed creation of the event resource.               |
| **2. Event Broadcast**  | EMS → Kafka (`event_created`).                   | **Asynchronous Decoupling:** EMS completes its work without waiting for downstream services. |
| **3. Fan-out Notification** | RNS → Kafka → USS (REST: get followers).       | **Synchronous + Asynchronous:** RNS performs a critical synchronous lookup to enable fan-out via non-blocking WebSockets. |
| **4. Discovery Update** | DRS → Kafka.                                     | **Data Independence:** DRS updates its internal MongoDB model to ensure the new event appears in future recommendations. |

---

### B. Key Feature Highlights

- **User Authentication & Profiles:** Managed securely by the dedicated User & Social Service (USS).
- **One-Way Following:** Enables students to subscribe to updates from organizations without reciprocal approval.
- **Real-Time Updates:** Users receive instant push notifications via WebSockets when followed organizations update an event's time or location.
- **Personalized Feeds:** The Discovery & Recommendation Service (DRS) uses user behavior data (views, follows) consumed from Kafka to deliver tailored event recommendations.


## 4. Technology Deep Dive (Tech Stack)

This project uses a Polyglot Persistence strategy and modern cloud-native tools:

| **Category**         | **Technology**          | **Usage in Project**                                                                 |
|-----------------------|-------------------------|-------------------------------------------------------------------------------------|
| **Backend Language**  | Not decided       | NA.                       |
| **Message Queue**     | Apache Kafka            | Central nervous system; ensures durable, ordered, and high-throughput communication across services. |
| **Relational DB**     | PostgreSQL              | Source of truth for structured, critical data (Users, Events).                     |
| **NoSQL DB / Cache**  | MongoDB                 | Used by DRS for flexible storage of recommendation model features.                 |
| **In-Memory Cache**   | Redis                   | Used by RNS to map active WebSocket connections (high-speed lookup).               |
| **Real-Time Layer**   | WebSockets              | The protocol used by the RNS for persistent, low-latency push notifications.        |
| **API Entry**         | Nginx / Kong            | Serves as the API Gateway layer for centralized security and routing.              |