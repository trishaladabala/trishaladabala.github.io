---
layout: page
title: Supermarket POS & Inventory Management System
description: Modular desktop retail automation system using JavaFX, MySQL, JDBC, and test-driven development with JUnit and Mockito.
importance: 1
category: software engineering & systems
related_publications: false
---

### Project Overview

Retail point-of-sale (POS) and inventory systems demand strict transactional integrity, high responsiveness, and zero tolerance for data anomalies such as phantom inventory or billing discrepancies. 

This project is a full-featured desktop retail management suite developed in **Java** and **JavaFX**, following strict **Model-View-Controller (MVC)** architectural design patterns. It integrates a relational **MySQL** database via **JDBC** and emphasizes rigorous unit and integration testing via **JUnit** and **Mockito**.

---

### Core Architecture & Engineering Highlights

```
+-----------------------------------------------------------+
|                        JavaFX UI                          |
|             (Views: FXML + CSS Layouts)                   |
+-----------------------------+-----------------------------+
                              | Event Actions / UI Bindings
                              v
+-----------------------------------------------------------+
|                        Controllers                        |
|       (Input Validation, Session State, Flow Control)     |
+-----------------------------+-----------------------------+
                              | Service Calls
                              v
+-----------------------------------------------------------+
|                       Service Layer                       |
|       (Billing Engine, Loyalty Calculator, Inventory Sync)|
+-----------------------------+-----------------------------+
                              | DAO Pattern / JDBC
                              v
+-----------------------------------------------------------+
|                      MySQL Database                       |
|          (Relational Schemas, Constraints, ACID)          |
+-----------------------------------------------------------+
```

#### 1. MVC Separation & Modular Design
- **View Layer**: Reactive JavaFX scenes designed with FXML and custom modern stylesheets, providing cashier billing screens, stock manager dashboards, and reporting views.
- **Controller Layer**: Decoupled handlers managing UI input sanitization, background worker tasks (`Task`/`Service`), and event coordination.
- **Model / DAO Layer**: Strongly typed POJOs with clean Data Access Object (DAO) interfaces executing prepared statements and managing relational mappings.

#### 2. Transaction Management & Persistence
- Utilized **JDBC** with explicit transactional boundaries (`commit` and `rollback`) to guarantee **ACID** properties during multi-item checkout operations.
- Enforced database constraints (foreign keys, cascading updates, unique SKU indexes) to ensure reliable inventory tracking even under rapid checkout sessions.

#### 3. Core Functional Capabilities
- **High-Throughput Billing Engine**: Rapid barcode/SKU item lookup, multi-tier tax computations, discounts, and formatted invoice generation.
- **Real-Time Inventory Tracking**: Automated stock decrementing upon transaction completion, low-stock alerts, and restocking order logging.
- **Customer Loyalty System**: Tiered membership rewards and point accrual calculated dynamically based on purchase volume.
- **Sales Analytics**: Aggregated daily, weekly, and monthly revenue metrics with visual summary charts.

#### 4. Automated Testing & Verification
- Authored extensive unit tests with **JUnit** targeting complex business logic in tax calculations, discount combinations, and inventory validation.
- Utilized **Mockito** to mock external DAO dependencies and database connections, enabling isolated, deterministic testing of the service layer with high test coverage.
