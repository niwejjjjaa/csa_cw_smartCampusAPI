# 🏫 Smart Campus — Sensor & Room Management REST API

**Module:** 5COSC022W – Client-Server Architectures  
**Stack:** JAX-RS (Jersey 2.41) · Java 11 · Maven · Apache Tomcat 9

---

## Student Information

| | |
|---|---|
| **Name** | [Niweja Weerasinghe] |
| **UoW Number** | [w2120305] |
| **IIT Number** | [20231459] |

---

## What This Project Does

This is a RESTful backend API that manages the physical infrastructure of a university smart campus — specifically Rooms and the Sensors installed inside them.

Campus administrators can use this API to:
- Add and remove rooms across the campus
- Register sensors (CO2, Temperature, Humidity, etc.) and link them to rooms
- Post sensor readings and track the full reading history per sensor
- Filter sensors by type
- Safely prevent rooms from being deleted while sensors are still assigned to them

All data is held in memory using `HashMap` and `ArrayList`. No database is involved.

---

## Base URL

```
http://localhost:8080/smart-campus-api/api/v1
```

---

## Endpoints at a Glance

| Method | Path | What it does |
|--------|------|--------------|
| GET | `/` | API discovery — metadata + links |
| GET | `/rooms` | List all rooms |
| POST | `/rooms` | Create a new room |
| GET | `/rooms/{id}` | Get one room by ID |
| DELETE | `/rooms/{id}` | Delete a room (must be empty) |
| GET | `/sensors` | List all sensors (optional `?type=` filter) |
| POST | `/sensors` | Register a new sensor |
| GET | `/sensors/{id}/readings` | Get reading history for a sensor |
| POST | `/sensors/{id}/readings` | Add a new reading to a sensor |

---

## Error Responses

The API never exposes raw stack traces. Every error returns a structured JSON body.

| Status | Cause |
|--------|-------|
| 400 | Missing required field (e.g. no room ID supplied) |
| 403 | Sensor is in MAINTENANCE — cannot accept readings |
| 404 | Requested resource does not exist |
| 409 | Room still has sensors assigned — cannot delete |
| 415 | Wrong Content-Type sent (must be `application/json`) |
| 422 | Sensor references a roomId that does not exist |
| 500 | Unexpected server error — generic safe message returned |

---

## How to Build and Run

### Prerequisites
- Java 11 or higher
- Maven 3.6+
- Apache Tomcat 9

### 1 — Get the code
```bash
git clone https://github.com/YOUR_USERNAME/smart-campus-api.git
cd smart-campus-api
```

### 2 — Build
```bash
mvn clean package
```
The WAR file will be created at `target/smart-campus-api.war`.

### 3 — Deploy
```bash
cp target/smart-campus-api.war /path/to/tomcat/webapps/
```

### 4 — Start the server
**Mac / Linux:**
```bash
/path/to/tomcat/bin/startup.sh
```
**Windows:**
```
\tomcat\bin\startup.bat
```

### 5 — Test it
```bash
curl http://localhost:8080/smart-campus-api/api/v1
```
You should get back a JSON object with API metadata. The server is running.

---

## Sample curl Commands

**1. Hit the discovery endpoint**
```bash
curl http://localhost:8080/smart-campus-api/api/v1
```

**2. Create a room**
```bash
curl -X POST http://localhost:8080/smart-campus-api/api/v1/rooms \
  -H "Content-Type: application/json" \
  -d "{\"id\":\"LIB-301\",\"name\":\"Library Quiet Study\",\"capacity\":40}"
```

**3. Register a CO2 sensor in that room**
```bash
curl -X POST http://localhost:8080/smart-campus-api/api/v1/sensors \
  -H "Content-Type: application/json" \
  -d "{\"id\":\"CO2-001\",\"type\":\"CO2\",\"status\":\"ACTIVE\",\"currentValue\":0,\"roomId\":\"LIB-301\"}"
```

**4. Post a reading to the sensor**
```bash
curl -X POST http://localhost:8080/smart-campus-api/api/v1/sensors/CO2-001/readings \
  -H "Content-Type: application/json" \
  -d "{\"value\":450.5,\"timestamp\":0}"
```

**5. Get all reading history for that sensor**
```bash
curl http://localhost:8080/smart-campus-api/api/v1/sensors/CO2-001/readings
```

**6. Filter sensors by type**
```bash
curl "http://localhost:8080/smart-campus-api/api/v1/sensors?type=CO2"
```

**7. Try to delete a room that still has sensors (expect 409)**
```bash
curl -X DELETE http://localhost:8080/smart-campus-api/api/v1/rooms/LIB-301
```

**8. Try to register a sensor with a fake roomId (expect 422)**
```bash
curl -X POST http://localhost:8080/smart-campus-api/api/v1/sensors \
  -H "Content-Type: application/json" \
  -d "{\"id\":\"TEMP-99\",\"type\":\"Temperature\",\"status\":\"ACTIVE\",\"currentValue\":0,\"roomId\":\"FAKE-ROOM\"}"
```

---

## Project Structure

```
smart-campus-api/
├── pom.xml
├── README.md
├── REPORT.md
├── postman/
│   └── Smart_Campus_API.json
└── src/main/java/com/smartcampus/app/
    ├── SmartCampusApplication.java       ← JAX-RS app entry point (@ApplicationPath)
    ├── api/
    │   ├── DiscoveryResource.java        ← GET /api/v1
    │   ├── SensorRoomResource.java       ← /rooms endpoints
    │   ├── SensorResource.java           ← /sensors endpoints
    │   └── SensorReadingResource.java    ← /sensors/{id}/readings (sub-resource)
    ├── model/
    │   ├── Room.java
    │   ├── Sensor.java
    │   └── SensorReading.java
    ├── store/
    │   └── CampusData.java               ← Shared in-memory data store
    ├── error/
    │   ├── RoomNotEmptyException.java
    │   ├── RoomNotEmptyExceptionMapper.java
    │   ├── LinkedResourceNotFoundException.java
    │   ├── LinkedResourceNotFoundExceptionMapper.java
    │   ├── SensorUnavailableException.java
    │   ├── SensorUnavailableExceptionMapper.java
    │   ├── NotFoundExceptionMapper.java
    │   └── GlobalExceptionMapper.java
    └── filter/
        └── RequestResponseLoggingFilter.java
```

---

## Postman Collection

Import `postman/Smart_Campus_API.json` into Postman to test all endpoints with pre-built requests.

---

## Report

See `REPORT.md` for answers to all coursework questions.
