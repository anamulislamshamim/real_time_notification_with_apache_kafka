# Notification Service Demonstration (`kafka-notify`)

This document outlines the steps to run and test a simple notification service that uses a **producer-consumer model** likely leveraging **Apache Kafka** (or a similar message queue) for asynchronous processing.

The setup involves three main components: a **Producer API** (handling the `/send` endpoint), a **Consumer** (processing messages from the queue), and a **Retrieval API** (handling the `/notifications/{userID}` endpoint).

-----

## Prerequisites

To run this demonstration, you will need:

  * **Go** installed on your system.
  * **Apache Kafka** (or a mock/embedded Kafka) and a service (like ZooKeeper) running and accessible on their default ports.
  * The source code for the `kafka-notify` project.
  * Three separate terminal windows.

-----

## Running the Demo

Follow these steps in the specified order across your three terminals.

### 1\. Start the Producer (API Gateway)

The producer starts the HTTP API that accepts new notification requests and sends them to the message queue (e.g., Kafka). It runs on port **8080**.

| Terminal | Command |
| :--- | :--- |
| **Terminal 1** | `cd kafka-notify` |
| | `go run cmd/producer/producer.go` |

### 2\. Start the Consumer (Worker)

The consumer runs in the background, listening to the message queue, processing the incoming notifications, and persisting them (likely to a database or in-memory store). This service also handles notification retrieval on port **8081**.

| Terminal | Command |
| :--- | :--- |
| **Terminal 2** | `cd kafka-notify` |
| | `go run cmd/consumer/consumer.go` |

-----

## Testing the Flow

With both services running, use a third terminal to simulate sending and retrieving notifications.

### 3\. Sending Notifications (Producer API - Port 8080)

Use the following `curl` commands to post data to the Producer API.

#### User 1 (Emma) Notifications

```bash
# Bruno started following Emma
curl -X POST http://localhost:8080/send -d "fromID=2&toID=1&message=Bruno started following you."

# Lena liked Emma's post
curl -X POST http://localhost:8080/send -d "fromID=4&toID=1&message=Lena liked your post: 'My weekend getaway!'"
```

#### User 2 (Bruno) Notification

```bash
# Emma mentioned Bruno in a comment
curl -X POST http://localhost:8080/send -d "fromID=1&toID=2&message=Emma mentioned you in a comment: 'Great seeing you yesterday, @Bruno!'"
```

### 4\. Retrieving Notifications (Retrieval API - Port 8081)

Fetch the stored notifications for a specific user ID.

#### Retrieving notifications for User 1 (Emma)

```bash
curl http://localhost:8081/notifications/1
```

**Expected Output (Example):**

```json
{"notifications": [
    {"from": {"id": 2, "name": "Bruno"}, "to": {"id": 1, "name": "Emma"}, "message": "Bruno started following you."},
    {"from": {"id": 4, "name": "Lena"}, "to": {"id": 1, "name": "Emma"}, "message": "Lena liked your post: 'My weekend getaway!'"}
]}
```

#### Retrieving notifications for User 2 (Bruno)

```bash
curl http://localhost:8081/notifications/2
```

**Expected Output (Example):**

```json
{"notifications": [
    {"from": {"id": 1, "name": "Emma"}, "to": {"id": 2, "name": "Bruno"}, "message": "Emma mentioned you in a comment: 'Great seeing you yesterday, @Bruno!'"}
]}
```