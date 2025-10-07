# Kafka Workflow in a Ride-Sharing App 🚗

## 1. Passenger Requests a Ride 📱
- **Producer**: Rider App  
- **Action**: Sends a ride request to Kafka (topic: `ride-requests`).
- **Topic**: `ride-requests`
  
    ```
    +------------+           +----------------------+           +-------------------+
    | Rider App  |  --->     | Kafka Topic:         |  --->     | Backend Service   |
    | (Producer) |           | ride-requests        |           | (Consumer)        |
    +------------+           +----------------------+           +-------------------+
    ```

- **Database Storage (MongoDB)**:
  - After ride matching, the **ride request** with the assigned driver will be stored in MongoDB  (`rides` collection) once the consumer processes it.


## 2. Location Updates from Drivers 📍
- **Producer**: Driver App  
- **Action**: Sends continuous GPS location updates to Kafka (topic: `driver-location-updates`).
- **Topic**: `driver-location-updates`

    ```
    +-------------+          +----------------------------+          +-------------------+
    | Driver App  |  --->    | Kafka Topic:               |  --->    | Backend Service   |
    | (Producer)  |          | driver-location-updates    |          | (Consumer)        |
    +-------------+          +----------------------------+          +-------------------+
    ```
    

- **In-Memory / Cache**:
  - Real-time location updates stored in an in-memory cache  like Redis for fast ETA and distance calculations  without  querying the DB.

- **Database Storage (MongoDB)**:
  - Periodic snapshots or final ride locations are stored in MongoDB for audit, analytics, and post-ride summaries.


## 3. Matching Service 🔍
- **Consumer**: Matching backend service
- **Action**: Listens to both `ride-requests` and `driver-location-updates` topics.
- **Action**: Matches rider with nearby driver.
- **Output**: Sends ride assignment info to Kafka (topic: `ride-assignments`).

   ``` 
    Kafka Topic: ride-requests -> Backend Service (Matching Service)

    Kafka Topic: driver-location-updates -> Backend Service (Matching)

    Backend Service -> Kafka Topic: ride-assignments -> Rider & Driver Apps (Consumers)
   ```

- **In-Memory / Cache**:
  - The matching service temporarily stores Real-time location data in Redis  to match driver and rider locations quickly without querying the DB.

- **Database Storage (MongoDB)**:
    - Once a driver is assigned, the ride info is persisted in MongoDB (rides collection) with the rider, driver, pickup location, and status.

## 4. Ride Confirmation ✅
- **Producer**: Backend service (ride matching result)
- **Action**: Sends ride assignment confirmation to Kafka (topic: `ride-assignments`).
- **Consumer**: passenger app, Driver app
- **Action**: Both apps consume this info and show confirmation.

    ```
    +------------------+      +--------------------------+      +-------------------------+
    | Backend Service  | ---> | Kafka Topic:             | ---> | Rider & Driver Apps     |
    | (Producer)       |      | ride-assignments         |      | (Consumers)             |
    +------------------+      +--------------------------+      +-------------------------+
    ```

- **Database Storage (MongoDB)**:
  - Data already stored during matching; this step is mainly a real-time notification.


## 5. Live Ride Tracking 🛣️
- **Producer**: Driver app (during the ride) 
- **Action**: Sends continuous real-time ride location updates to Kafka (`live-ride-tracking`).  
- **Consumers**: Other services (ETA , Map , Distance remaining , customer support, etc.)
- **Action**: Services consume the real-time data.  

    ```
    +-------------+          +---------------------------+          +------------------------------+
    | Driver App  |  --->    | Kafka Topic:              |  --->    | ETA / Map / Support Services |
    | (Producer)  |          | live-ride-tracking        |          | (Consumers)                  |
    +-------------+          +---------------------------+          +------------------------------+
    ```

- **In-Memory / Cache**:
  - Real-time location updates are stored in-memory (via Redis) to calculate ETA, distance remaining, and support live map tracking for the rider and driver.

- **Database Storage (MongoDB)**:
   - MongoDB may store periodic snapshots of location, particularly for audit or dispute purposes (e.g., in ride_logs or driver_locations collections).


## 6. Payment & Ratings 💵
- **Producers**: Rider App, Driver App  
- **Action**: After ride ends, sends payment info and ratings to Kafka (`ride-completions`, `payments`).  
- **Consumers**: Billing, Analytics, User Profile Services  
- **Action**: These services consume the data for processing.

    ```
    +-----------------------+     +---------------------------------+     +-----------------------------+
    | Rider/Driver Apps     | --> | Kafka Topics:                   | --> | Billing / Analytics Services|
    | (Producers)           |     | ride-completions, payments      |     | (Consumers)                 |
    +-----------------------+     +---------------------------------+     +-----------------------------+
    ```

- **Database Storage (MongoDB)**:
   - After processing, payment details , ratings, and ride summaries are stored in MongoDB for user profiles and analytics purposes (collections like payments, ratings, and user_profiles).


---

## Data Flow Summary 🔁
```
Apps (Producers) → Kafka Topics → Backend Services (Consumers)
```

| Type | Components |
|------|-------------|
| **Producers** | Rider App, Driver App, Backend Services |
| **Kafka Topics** | ride-requests, driver-location-updates, ride-assignments, live-ride-tracking, ride-completions, payments |
| **Consumers** | Backend Services (Matching, ETA, Billing, Analytics), Rider & Driver Apps |

- **In-Memory / Cache**:  
  Redis or similar used to store real-time GPS locations (driver/rider) for ETA, distance remaining, and live tracking.
  Temporary storage for fast processing of location data, without querying the database constantly.

- **Database (MongoDB)**:  
  After finalizing a step (ride assignment, payment, or ride completion), persistent data is stored in MongoDB for audit, analytics, and historical purposes.
  Collections like rides, payments, ride_logs, ratings, user_profiles.

> 🧩 **Kafka acts as the real-time data pipeline (backbone)**, ensuring fast, reliable communication between apps and services without overloading the database.
