# 🚗 Vehicle Rental System - Database Design & SQL Queries

## 📋 Project Overview

This project focuses on designing a relational database for a Vehicle Rental System and writing essential SQL queries based on real-world business requirements.
The project evaluates understanding of ERD design, table relationships, primary & foreign keys, and SQL querying techniques.

---

## 🎯 Objectives

- Design an ERD with 1:1, 1:Many, and Many:1 relationships
- Understand primary keys and foreign keys
- Write SQL queries using JOIN, EXISTS, and WHERE clauses
- Implement business logic through database constraints

---

## 🏗️ Database Design

Entity Relationship Diagram (ERD)

```ts
USERS (1) ──── makes ────► (M) BOOKINGS (M) ──── books ────► (1) VEHICLES
```

## SQL Queries

### CREATE DATABASE

```ts
CREATE DATABASE VehicleRentalSystem
```

### CREATE Users TABLE

```ts
CREATE TABLE Users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    phone VARCHAR(20),
    role VARCHAR(20) CHECK (role IN ('Admin', 'Customer')) NOT NULL
);
```

### INSERT Users

```ts
INSERT INTO Users (user_id, name, email, phone, role) VALUES
(1, 'Alice', 'alice@example.com', '1234567890', 'Customer'),
(2, 'Bob', 'bob@example.com', '0987654321', 'Admin'),
(3, 'Charlie', 'charlie@example.com', '1122334455', 'Customer');
```

### SELECT All Users

```ts
SELECT * FROM Users
```

### Users Table

| user_id | name    | email               | phone      | role     |
| :------ | :------ | :------------------ | :--------- | :------- |
| 1       | Alice   | alice@example.com   | 1234567890 | Customer |
| 2       | Bob     | bob@example.com     | 0987654321 | Admin    |
| 3       | Charlie | charlie@example.com | 1122334455 | Customer |

### CREATE Vehicles TABLE

```ts
CREATE TABLE Vehicles (
    vehicle_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) CHECK (type IN ('car', 'bike', 'truck')) NOT NULL,
    model VARCHAR(20) NOT NULL,
    registration_number VARCHAR(50) UNIQUE NOT NULL,
    rental_price NUMERIC(10, 2) NOT NULL CHECK (rental_price > 0),
    status VARCHAR(20) CHECK (status IN ('available', 'rented', 'maintenance')) NOT NULL
);
```

### INSERT Vehicles

```ts
INSERT INTO Vehicles (
    vehicle_id, name, type, model, registration_number, rental_price, status
) VALUES
(1, 'Toyota Corolla', 'car', 2022, 'ABC-123', 50, 'available'),
(2, 'Honda Civic', 'car', 2021, 'DEF-456', 60, 'rented'),
(3, 'Yamaha R15', 'bike', 2023, 'GHI-789', 30, 'available'),
(4, 'Ford F-150', 'truck', 2020, 'JKL-012', 100, 'maintenance');
```

### SELECT All Vehicles

```ts
SELECT * FROM Vehicles
```

### Vehicles Table

| vehicle_id | name           | type  | model | registration_number | rental_price | status      |
| :--------- | :------------- | :---- | :---- | :------------------ | :----------- | :---------- |
| 1          | Toyota Corolla | car   | 2022  | ABC-123             | 50           | available   |
| 2          | Honda Civic    | car   | 2021  | DEF-456             | 60           | rented      |
| 3          | Yamaha R15     | bike  | 2023  | GHI-789             | 30           | available   |
| 4          | Ford F-150     | truck | 2020  | JKL-012             | 100          | maintenance |

### CREATE Bookings TABLE

```ts
CREATE TABLE Bookings (
    booking_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    vehicle_id INT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status VARCHAR(20) CHECK (status IN ('pending', 'confirmed', 'completed', 'cancelled')) NOT NULL,
    total_cost NUMERIC(10, 2) NOT NULL,

    CONSTRAINT fk_booking_user
        FOREIGN KEY (user_id)
        REFERENCES Users(user_id)
        ON DELETE CASCADE,

    CONSTRAINT fk_booking_vehicle
        FOREIGN KEY (vehicle_id)
        REFERENCES Vehicles(vehicle_id)
        ON DELETE CASCADE
);
```

### INSERT Bookings

```ts
INSERT INTO Bookings (
    booking_id, user_id, vehicle_id, start_date, end_date, status, total_cost
) VALUES
(1, 1, 2, '2023-10-01', '2023-10-05', 'completed', 240),
(2, 1, 2, '2023-11-01', '2023-11-03', 'completed', 120),
(3, 3, 2, '2023-12-01', '2023-12-02', 'confirmed', 60),
(4, 1, 1, '2023-12-10', '2023-12-12', 'pending', 100);
```

### SELECT All bookings

```ts
SELECT * FROM bookings
```

### Bookings Table

| booking_id | user_id | vehicle_id | start_date | end_date   | status    | total_cost |
| :--------- | :------ | :--------- | :--------- | :--------- | :-------- | :--------- |
| 1          | 1       | 2          | 2023-10-01 | 2023-10-05 | completed | 240        |
| 2          | 1       | 2          | 2023-11-01 | 2023-11-03 | completed | 120        |
| 3          | 3       | 2          | 2023-12-01 | 2023-12-02 | confirmed | 60         |
| 4          | 1       | 1          | 2023-12-10 | 2023-12-12 | pending   | 100        |

---

## Expected Query Results

### Query 1: JOIN

**Requirement**: Retrieve booking information along with Customer name and Vehicle name.

```ts
SELECT
    b.booking_id,
    u.name AS customer_name,
    v.name AS vehicle_name,
    b.start_date,
    b.end_date,
    b.status
FROM Bookings AS b
INNER JOIN Users AS u ON b.user_id = u.user_id
INNER JOIN Vehicles AS v ON b.vehicle_id = v.vehicle_id
```

**Expected Output**:
| booking_id | customer_name | vehicle_name | start_date | end_date | status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Alice | Honda Civic | 2023-10-01 | 2023-10-05 | completed |
| 2 | Alice | Honda Civic | 2023-11-01 | 2023-11-03 | completed |
| 3 | Charlie | Honda Civic | 2023-12-01 | 2023-12-02 | confirmed |
| 4 | Alice | Toyota Corolla | 2023-12-10 | 2023-12-12 | pending |

---

### Query 2: EXISTS

**Requirement**: Find all vehicles that have never been booked.

```ts
SELECT
    v.vehicle_id,
    v.name,
    v.type,
    v.model,
    v.registration_number,
    ROUND(v.rental_price) AS rental_price,
    v.status
FROM Vehicles AS v
WHERE NOT EXISTS (
    SELECT 1
    FROM Bookings AS b
    WHERE b.vehicle_id = v.vehicle_id
)
ORDER BY v.vehicle_id;
```

**Expected Output**:
| vehicle_id | name | type | model | registration_number | rental_price | status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 3 | Yamaha R15 | bike | 2023 | GHI-789 | 30 | available |
| 4 | Ford F-150 | truck | 2020 | JKL-012 | 100 | maintenance |

---

### Query 3: WHERE

**Requirement**: Retrieve all available vehicles of a specific type (e.g. cars).

```ts
SELECT
    vehicle_id,
    name,
    type,
    model,
    registration_number,
    ROUND(rental_price) AS rental_price,
    status
FROM Vehicles
WHERE type = 'car'
  AND status = 'available'
```

**Expected Output**:
| vehicle_id | name | type | model | registration_number | rental_price | status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Toyota Corolla | car | 2022 | ABC-123 | 50 | available |

---

### Query 4: GROUP BY and HAVING

**Requirement**: Find the total number of bookings for each vehicle and display only those vehicles that have more than 2 bookings.

```ts
  SELECT
    v.name AS vehicle_name,
    COUNT(b.booking_id) AS total_bookings
FROM Bookings AS b
INNER JOIN Vehicles AS v ON b.vehicle_id = v.vehicle_id
GROUP BY v.name
HAVING COUNT(b.booking_id) > 2
```

**Expected Output**:
| vehicle_name | total_bookings |
| :--- | :--- |
| Honda Civic | 3 |

---

## Part 3: Theory Questions (Viva Practice - Progress, Not Perfection)

#### Question 1

What is a foreign key and why is it important in relational databases?

#### Question 2

What is the difference between WHERE and HAVING clauses in SQL?

#### Question 3

What is a primary key and what are its characteristics?

#### Question 4

What is the difference between INNER JOIN and LEFT JOIN in SQL?

I have completed the **Theory Questions**.

👉 **Theory Questions Video Link:**  
[Click here to watch the video](https://drive.google.com/file/d/1ZytNbX9ctCSP2R4XKvL3KgFwB0cXnBIk/view?usp=sharing)

---

## 🛠️ Tools & Technologies

- Database: PostgreSQL
- ERD Tool: Lucidchart
- Query Language: SQL

## 🔗 Project Links

- 📁 **GitHub Repository:**  
  [Vehicle Rental System – Database Design & SQL Queries](https://github.com/abdulbariks/Vehicle-Rental-System---Database-Design-SQL-Queries)

- 📊 **ERD (Lucidchart):**  
  [View ERD Diagram](https://lucid.app/lucidchart/08a1daf6-29dd-4492-ace3-e2798d98d658/edit?viewport_loc=-1418%2C-64%2C2218%2C1022%2C0_0&invitationId=inv_0cd077d4-222e-4305-bf5f-b2031e8cc37b)

- 🎥 **Viva / Theory Questions Video:**  
  [Watch Video](https://drive.google.com/file/d/1ZytNbX9ctCSP2R4XKvL3KgFwB0cXnBIk/view?usp=sharing)

---
