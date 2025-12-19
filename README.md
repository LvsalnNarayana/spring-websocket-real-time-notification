
# Spring-Websocket-Real-Time-Notification

## Overview

This project is a **multi-microservice demonstration** of **real-time notifications** using **WebSocket** with **STOMP over WebSocket** in **Spring Boot 3.x**. It showcases how to build scalable, bidirectional communication for instant updates — a core requirement in modern interactive applications.

The focus is on **production-grade WebSocket patterns**: message brokering, user destinations, presence tracking, authentication, error handling, and scaling considerations.

## Real-World Scenario (Simulated)

In chat and messaging apps like **WhatsApp**, **Slack**, or **Telegram**:
- Users receive instant message notifications, typing indicators, and read receipts.
- Thousands of concurrent connections must be managed efficiently.
- Messages are delivered to specific users or groups in real-time.
- Connection state (online/offline) needs to be tracked.

We simulate a real-time notification system where users connect via WebSocket, subscribe to personal or group topics, and receive instant updates (messages, alerts, presence changes) with full STOMP protocol support.

## Microservices Involved

| Service                     | Responsibility                                                                 | Port  |
|-----------------------------|--------------------------------------------------------------------------------|-------|
| **eureka-server**           | Service discovery (Netflix Eureka)                                             | 8761  |
| **notification-hub**        | WebSocket broker: handles STOMP connections, routing, and broadcasting        | 8080  |
| **chat-service**            | Business logic: manages chat rooms, messages, typing indicators                | 8081  |
| **presence-service**        | Tracks user online/offline status, broadcasts presence events                  | 8082  |

All real-time communication flows through the **notification-hub** using a message broker (in-memory or Redis for production).

## Tech Stack

- Spring Boot 3.x
- Spring WebSocket + STOMP
- Spring Messaging (SimpMessagingTemplate)
- Spring Security (WebSocket authentication)
- SockJS + STOMP client fallback
- Spring Cloud Netflix Eureka
- Redis (optional external broker for scaling)
- Micrometer + Actuator (WebSocket metrics)
- Lombok
- Maven (multi-module)
- Docker & Docker Compose

## Docker Containers

```yaml
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"

  notification-hub:
    build: ./notification-hub
    depends_on:
      - redis
      - eureka-server
    ports:
      - "8080:8080"

  chat-service:
    build: ./chat-service
    depends_on:
      - eureka-server
    ports:
      - "8081:8081"

  presence-service:
    build: ./presence-service
    depends_on:
      - eureka-server
    ports:
      - "8082:8082"
```

Run with: `docker-compose up --build`

## WebSocket & STOMP Features

| Feature                        | Implementation Details                                                  |
|--------------------------------|-------------------------------------------------------------------------|
| **STOMP over WebSocket**       | `/ws` endpoint with SockJS fallback                                     |
| **User Destinations**          | `/user/{sessionId}/queue/*` and `/user/queue/*`                         |
| **Topic Broadcasting**         | `/topic/chat.room.{id}` for group messages                              |
| **Presence Tracking**          | Subscribe to `/topic/presence`, CONNECT/DISCONNECT events               |
| **Authentication**             | Spring Security + ChannelInterceptor for user principal                 |
| **Message Mapping**            | `@MessageMapping` + `@SendTo` / `SimpMessagingTemplate`                 |
| **Error Handling**             | `StompExceptionHandler` with `@ControllerAdvice`                       |
| **Scaling**                    | In-memory broker or Redis (configurable)                                |

## Key Features

- Full STOMP protocol support (connect, subscribe, send, ack)
- Private messaging via user destinations
- Group chat with topic broadcasting
- Real-time presence (online/offline/typing)
- Authenticated WebSocket connections
- Heartbeat and connection management
- Graceful disconnect handling
- Metrics: active sessions, messages/sec
- Client fallback with SockJS
- Redis broker option for horizontal scaling

## Expected Endpoints

### Notification Hub (`ws://localhost:8080/ws`)

**STOMP Destinations**:
- Connect: `/ws`
- Subscribe private: `/user/queue/notifications`
- Subscribe group: `/topic/chat.room.{roomId}`
- Subscribe presence: `/topic/presence`
- Send message: `/app/chat.send` (payload → roomId, content)
- Typing indicator: `/app/chat.typing`

### Chat Service (`http://localhost:8081`)

| Method | Endpoint                        | Description                                      |
|--------|---------------------------------|--------------------------------------------------|
| GET    | `/api/rooms`                    | List available chat rooms                        |
| POST   | `/api/rooms/{id}/join`          | Join room (triggers presence)                    |

### Presence Service (`http://localhost:8082`)

| Method | Endpoint                        | Description                                      |
|--------|---------------------------------|--------------------------------------------------|
| GET    | `/api/users/online`             | List currently online users                      |

### Sample Client (HTML/JS)

Provided `client/index.html` — simple chat UI with:
- Connect with username
- Join rooms
- Send/receive messages
- See typing and presence

## Architecture Overview

```
Clients (Browser/Mobile)
   ↓ (WebSocket + STOMP)
Notification Hub
   ├── Auth → Spring Security
   ├── CONNECT → Presence Service (online)
   ├── SUBSCRIBE → /topic/* or /user/queue/*
   ├── SEND → /app/* → @MessageMapping in Chat Service
   └── DISCONNECT → Presence Service (offline)
   ↓
Redis Broker (optional for scaling)
```

**Message Flow**:
1. Client connects → authenticated → session tracked
2. Subscribe to `/user/queue/notifications`
3. Send message → routed to `/app/chat.send` → processed → broadcast to `/topic/chat.room.X`
4. Recipients receive instantly

## How to Run

1. Clone repository
2. Start Docker: `docker-compose up --build`
3. Access Eureka: `http://localhost:8761`
4. Open client: `http://localhost:8080/client/index.html`
5. Multiple browser tabs → connect with different usernames
6. Join room → see presence
7. Chat → instant delivery
8. Check metrics → active WebSocket sessions

## Testing Real-Time

1. Multiple clients → messages appear instantly
2. Typing indicator → real-time
3. Close tab → others see "offline"
4. Scale hub instances (with Redis) → connections persist
5. Unauthorized connect → rejected

## Skills Demonstrated

- Full WebSocket + STOMP implementation
- Real-time bidirectional communication
- User-specific and topic-based messaging
- Presence and lifecycle management
- Secure WebSocket connections
- Scaling considerations (Redis broker)
- Client fallback and compatibility
- Production monitoring of WebSocket sessions

## Future Extensions

- Redis external broker for multi-instance scaling
- Message persistence and history
- Push notifications (fallback when offline)
- End-to-end encryption
- Rate limiting per user
- Clustering with Sticky Sessions
- Integration with Kafka for durability

