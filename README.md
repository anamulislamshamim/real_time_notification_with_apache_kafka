# Notification Service Demonstration (real\_time\_notification\_with\_apache\_kafka) 🚀

This repository contains a simple notification service demonstrating a **real-time, asynchronous architecture** using a **producer-consumer model** powered by **Apache Kafka**.

The application consists of three main parts:

1.  **Producer API (Port 8080):** Accepts new notification requests and publishes them to Kafka.
2.  **Consumer (Worker):** Consumes messages from Kafka, processes them, and persists the data.
3.  **Retrieval API (Port 8081):** Fetches the stored notifications for a user.

-----

## Getting Started

### Prerequisites

  * **Go** installed on your system.
  * **Docker** and **Docker Compose** installed to run the Kafka environment.
  * **Git** for cloning the repository.

### Setup and Installation

Follow these steps to get the project and its dependencies running:

#### 1\. Clone the Repository

Open your terminal and clone the project:

```bash
git clone https://github.com/anamulislamshamim/real_time_notification_with_apache_kafka.git
```

#### 2\. Navigate to the Directory

```bash
cd real_time_notification_with_apache_kafka
```

#### 3\. Start Kafka (using Docker)

This project uses a Docker image for Kafka, which simplifies the setup. You do **not** need to manually start ZooKeeper; it is handled within the Docker configuration.

Run the following command to start the Kafka container in detached mode:

```bash
docker-compose up -d
```

-----

## Running the Demo

Once Kafka is running via Docker, you can start the Go services. You will need **three separate terminal windows** for the full demonstration.

### 1\. Start the Producer (API Gateway)

The Producer handles the `/send` endpoint and runs on port **8080**.

| Terminal | Command |
| :--- | :--- |
| **Terminal 1** | `go run cmd/producer/producer.go` |

### 2\. Start the Consumer (Worker & Retrieval API)

The Consumer listens to Kafka, processes messages, and hosts the retrieval API on port **8081**.

| Terminal | Command |
| :--- | :--- |
| **Terminal 2** | `go run cmd/consumer/consumer.go` |

-----

## Testing the Flow (Terminal 3)

With both services running, use **Terminal 3** to simulate the notification flow.

### 3\. Sending Notifications (Producer API - Port 8080)

Post data to the Producer API to send messages to the Kafka topic.

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

Fetch the stored notifications for a specific user ID after the consumer has had a moment to process the queue.

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

-----

## Cleanup

To stop the Kafka environment and remove the containers:

```bash
docker-compose down
```