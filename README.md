# Restful Booker — Performance Testing with Apache JMeter

## Project Overview

This project focuses on performance testing of the **Restful Booker** application using **Apache JMeter**.

The test simulates multiple concurrent users performing a realistic room booking workflow and measures key performance metrics such as response time, percentiles, throughput, and error rate.

---

## Objective

The main objective of this project is to evaluate the performance of the room booking workflow under different levels of concurrent load.

The following metrics were monitored:

* Average Response Time
* Median (P50)
* P90
* P95
* P99
* Minimum Response Time
* Maximum Response Time
* Error Percentage
* Throughput

> Note: No specific performance SLA was provided, so the results are reported objectively without classifying the system as passing or failing based on response-time thresholds.

---

## Tools & Technologies

* **Apache JMeter**
* **REST APIs**
* **JSON**
* **CSV Data Set Config**
* **Postman / Browser DevTools** for API investigation
* **GitHub** for project documentation

---

## Application

**Restful Booker / Automation in Testing**

The tested workflow is based on the room booking functionality.

---

## Business Scenario

The simulated user performs the following workflow:

1. Check room availability
2. Select a room
3. Create a booking

### Business Flow

```text
Check Room Availability
        ↓
Select Room
        ↓
Create Booking
```

---

## API Test Flow

### 1. Check Room Availability

**Method:** GET

```text
/api/room?checkin=${checkin}&checkout=${checkout}
```

The response is validated and the `roomid` is extracted dynamically.

---

### 2. View Selected Room

**Method:** GET

```text
/api/room/${roomid}
```

The previously extracted `roomid` is used dynamically.

---

### 3. Create Booking

**Method:** POST

```text
/api/booking
```

Example request body:

```json
{
  "roomid": ${roomid},
  "firstname": "${firstname}",
  "lastname": "${lastname}",
  "depositpaid": false,
  "email": "${email}",
  "phone": "${phone}",
  "bookingdates": {
    "checkin": "${checkin}",
    "checkout": "${checkout}"
  }
}
```

The response is validated and the dynamic `bookingid` is extracted.

---

# JMeter Test Plan Structure

```text
Test Plan
│
└── Thread Group
    │
    ├── HTTP Header Manager
    │
    ├── CSV Data Set Config
    │
    └── Book Room Transaction
        │
        ├── Check Room Availability
        │   ├── JSON JMESPath Assertion
        │   └── JSON JMESPath Extractor
        │
        ├── Think Time - Select Room
        │
        ├── View Selected Room
        │   └── JSON JMESPath Assertion
        │
        └── Create Booking
            ├── JSON JMESPath Assertion
            └── JSON JMESPath Extractor
```

---

## Parameterization

Test data was externalized into a CSV file using **CSV Data Set Config**.

The CSV contains dynamic booking information such as:

* Check-in date
* Check-out date
* First name
* Last name
* Email
* Phone number

Example:

```text
checkin,checkout,firstname,lastname,email,phone
2027-09-01,2027-09-02,Ahmed,Ali,ahmed01@test.com,01000000001
```

Using CSV data helps avoid hardcoding the same test data for every virtual user.

---

## Correlation

Dynamic values returned from one request were extracted and reused in subsequent requests.

### Example

The availability response returns a dynamic:

```text
roomid
```

This value is extracted using a **JSON JMESPath Extractor** and reused in:

```text
/api/room/${roomid}
```

Similarly, the booking response returns:

```text
bookingid
```

which was extracted dynamically.

This demonstrates API correlation in JMeter.

---

## Assertions

Assertions were added to validate the correctness of the API responses rather than relying only on HTTP status codes.

Examples include validating:

* Availability response contains rooms
* Selected room response contains the expected room ID
* Booking response contains a booking ID

This helps distinguish a technically successful HTTP request from an unsuccessful business response.

---

## Think Time

A **2-second Constant Timer** was added between selecting a room and viewing the selected room.

This represents a simplified user think time between actions.

The Think Time was excluded from the Transaction Controller's measured duration so that the transaction timing represents the application processing time rather than the simulated user delay.

---

# Performance Test Scenarios

The main transaction tested was:

```text
Book Room Transaction
```

The test was executed with different numbers of virtual users.

| Virtual Users |    Ramp-Up | Loop Count |
| ------------: | ---------: | ---------: |
|            10 | Controlled |          1 |
|            25 | Controlled |          1 |
|            50 | Controlled |          1 |
|           100 |    100 sec |          1 |

The same general test flow and configuration were maintained when comparing the isolated runs.

---

# Test Results

## 10 Users

| Metric  |    Result |
| ------- | --------: |
| Samples |        10 |
| Average | 562.70 ms |
| P50     | 521.50 ms |
| P90     | 931.90 ms |
| P95     |    971 ms |
| P99     |    971 ms |
| Min     |    473 ms |
| Max     |    971 ms |
| Error % |        0% |

---

## 25 Users — Repeated Run

The first 25-user run produced significantly higher response times, so the test was repeated under controlled conditions.

The repeated run produced:

| Metric  |  Result |
| ------- | ------: |
| Samples |      25 |
| Average |  521 ms |
| P50     |  495 ms |
| P90     |  537 ms |
| P95     |  607 ms |
| P99     | 1214 ms |
| Min     |  442 ms |
| Max     | 1214 ms |
| Error % |      0% |

The repeated run was used for comparison with the 50- and 100-user tests.

---

## 50 Users

| Metric  | Result |
| ------- | -----: |
| Samples |     50 |
| Average | 492 ms |
| P50     | 489 ms |
| P90     | 530 ms |
| P95     | 537 ms |
| P99     | 592 ms |
| Min     | 441 ms |
| Max     | 592 ms |
| Error % |     0% |

---

## 100 Users

| Metric  |  Result |
| ------- | ------: |
| Samples |     100 |
| Average |  503 ms |
| P50     |  490 ms |
| P90     |  528 ms |
| P95     |  543 ms |
| P99     |  755 ms |
| Min     |  429 ms |
| Max     | 1527 ms |
| Error % |      0% |

---

# Results Comparison

| Users | Average |    P50 |    P90 |    P95 |     P99 |     Max | Error % |
| ----: | ------: | -----: | -----: | -----: | ------: | ------: | ------: |
|    25 |  521 ms | 495 ms | 537 ms | 607 ms | 1214 ms | 1214 ms |      0% |
|    50 |  492 ms | 489 ms | 530 ms | 537 ms |  592 ms |  592 ms |      0% |
|   100 |  503 ms | 490 ms | 528 ms | 543 ms |  755 ms | 1527 ms |      0% |

---

# Key Observations

### 1. Response times remained relatively consistent

The average response time for the repeated 25-user, 50-user, and 100-user tests remained around the 500 ms range:

```text
25 Users → 521 ms
50 Users → 492 ms
100 Users → 503 ms
```

### 2. Median response time remained stable

P50 remained close to 500 ms across the three tests:

```text
25 Users → 495 ms
50 Users → 489 ms
100 Users → 490 ms
```

### 3. Tail latency increased at 100 users

The 100-user test recorded:

```text
P99 → 755 ms
Max → 1527 ms
```

This indicates that a small number of requests took considerably longer than the majority of transactions.

### 4. No errors were observed

All three comparison runs recorded:

```text
Error Rate = 0%
```

### 5. Repeated testing was important

The first 25-user run produced an average response time of approximately:

```text
1801 ms
```

while the repeated 25-user run produced:

```text
521 ms
```

This demonstrates why performance results should not be interpreted from a single isolated run.

---

# Important Test Limitation

The booking API modifies application state by creating bookings.

Therefore, test data availability and booking conflicts can affect test results.

For a production-level performance test, test data should be carefully controlled and isolated to ensure that functional failures caused by data conflicts are not incorrectly interpreted as performance failures.

---

# Conclusion

The project demonstrates a complete JMeter-based performance testing workflow for a REST API business scenario.

It covers:

* Virtual user simulation
* Load generation
* API performance measurement
* Parameterization
* Correlation
* Assertions
* Think Time
* Transaction measurement
* Percentile analysis
* Repeated performance testing
* Results comparison

The final 100-user test recorded:

```text
Average: 503 ms
P95:     543 ms
P99:     755 ms
Errors:  0%
```

These results are reported as observations because no predefined performance SLA was available for the tested application.

---

## Project Structure

```text
Restful-Booker-JMeter-Performance-Testing/
│
├── README.md
│
├── Test-Data/
│   └── booking_data.csv
│
├── JMeter/
│   └── Restful-Booker-Performance-Test.jmx
│
└── Results/
    └── Performance-Test-Results/
```

---

## Skills Demonstrated

* Performance Testing
* Apache JMeter
* REST API Testing
* HTTP Methods
* JSON
* Parameterization
* CSV Data Set Config
* Correlation
* JSON JMESPath Extractor
* JSON Assertions
* Transaction Controller
* Think Time
* Load Testing
* Response Time Analysis
* Percentile Analysis
* Throughput Analysis
* Error Rate Analysis
* Test Data Management
* Performance Test Result Interpretation
