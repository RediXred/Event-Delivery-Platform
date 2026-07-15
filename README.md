# Event-Delivery-Platform
Reliable event delivery platform that guarantees event delivery to external systems through queues, retries, idempotency, HMAC signing, delivery tracking, and comprehensive observability.

## How it works
The platform acts as an intermediary between event producers and event consumers.

### 1. Endpoint registration
Clients register one or more delivery endpoints and specify which event types they want to receive.

```text
Endpoint
    ├── URL
    ├── Event types
    └── Secret for request signing
```

### 2. Event ingestion
Applications publish events to the platform through the API. The platform immediately acknowledges the request without waiting for downstream systems to process the event.

### 3. Event persistence
The event is stored in the database and marked as accepted. At the same time, delivery tasks are created for all subscribed endpoints.

```text
Event
    ├── Delivery #1
    ├── Delivery #2
    ├── ...
    └── Delivery #N
```

Each delivery is processed independently.

### 4. Queueing
Delivery tasks are published to a message broker. Queues decouple event ingestion from delivery and allow horizontal scaling of workers.

### 5. Event delivery
Workers consume delivery tasks and send HTTP requests to the destination endpoints.

Each request contains:
    - event payload;
    - event identifier;
    - HMAC signature;
    - timestamp.

The HMAC signature allows receivers to verify the authenticity and integrity of the request.

### 6. Delivery result
If the receiver returns a successful response, the delivery is marked as completed. Otherwise, the delivery is marked as failed and scheduled for another attempt. All attempts are persisted for auditing and debugging purposes.

## Retry mechanism
The platform automatically retries failed deliveries using an exponential backoff algorithm.

## Dead Letter Queue (DLQ)
If a delivery fails, the platform moves it to a dead letter queue (DLQ) for further processing. The DLQ is a separate queue that is used to store events that have failed multiple times. Failed deliveries can later be:
    - replayed;
    - inspected;
    - manually retried;
    - removed.

## Idempotency
The platform guarantees **at-least-once delivery**. Because a delivery may be retried, receivers should process events idempotently using the event identifier.

## Transactional Outbox
To avoid losing events between database writes and message publication, the platform uses the Transactional Outbox pattern. This guarantees that accepted events will eventually be published and delivered.