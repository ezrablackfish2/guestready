# GuestReady – Hospitality Booking & Property Management Platform

GuestReady is a scalable hospitality booking and property-management platform inspired by modern short-term and mid-term rental services.

The project is designed around a **microservices architecture**, separating authentication, accommodation management, bookings, notifications, and customer reviews into independent services that can be developed, deployed, monitored, and scaled separately.

Rather than functioning as a simple property-listing application, the platform demonstrates the backend infrastructure required to operate a modern accommodation marketplace where guests can discover properties, create bookings, manage reservations, receive notifications, and review completed stays.

Property and platform administrators can manage hotels, rooms, room categories, users, permissions, bookings, and associated operational data through dedicated APIs.

## Overview

The system consists of five primary microservices:

* **Authentication Service**
* **Booking Service**
* **Hotel & Property Service**
* **Notification Service**
* **Review & Rating Service**

Each service owns a specific part of the application's business logic and communicates with other services through REST APIs and asynchronous messaging.

This separation allows individual components to scale independently while keeping the overall platform modular, maintainable, fault-tolerant, and easier to extend.

---

# Core Features

## 🔐 Authentication & User Management

The authentication service is built in Go and handles identity, account security, authorization, and protected access throughout the platform.

### Features

* User registration
* User login
* JWT authentication
* Password hashing with bcrypt
* Protected profile endpoints
* Request validation
* Rate limiting
* Database connection management
* Role-based access control
* Permission-based authorization
* Automatic default role assignment
* OTP email verification
* Password-reset OTP support
* Service proxying
* Standardized API responses
* Health checks

### OTP Verification

The platform includes a secure OTP system for account verification and password-related operations.

Features include:

* Cryptographically generated 6-digit OTP codes
* OTP expiration
* Single-use verification codes
* Automatic cleanup
* Email verification
* Password-reset support
* Rate limiting
* Input validation
* Integration with the Notification Service

---

# 🛡️ Role-Based Access Control

The authentication system implements granular role and permission management.

The platform can support roles such as:

* Guest
* User
* Property Manager
* Administrator

Permissions can be assigned to roles and roles can be assigned to individual users.

This makes it possible to protect sensitive platform operations and expose functionality according to the responsibilities of each user.

---

# 🏨 Property & Hotel Management

The Hotel Service manages accommodations, rooms, availability, categories, ratings, and related property information.

### Features

* Create properties
* Retrieve property information
* Update properties
* Soft-delete properties
* Search properties
* Filter properties
* Manage rooms
* Manage room categories
* Track room availability
* Store room pricing
* Property rating support
* Database migrations
* Repository-pattern architecture
* Request validation
* Error handling
* Structured logging
* Correlation ID tracking

The service uses soft deletes to preserve historical data rather than permanently removing property records.

---

# 🛏️ Room Management

Properties can contain multiple rooms and room categories.

Room information can include:

* Property association
* Room category
* Availability date
* Price
* Booking association

This structure allows the booking system to track accommodation inventory independently from the core property information.

---

# 📅 Booking Management

The Booking Service manages the complete reservation lifecycle.

### Features

* Create bookings
* Confirm bookings
* Cancel bookings
* Booking-status management
* Pending reservations
* Confirmed reservations
* Cancelled reservations
* Database transactions
* Idempotent booking operations
* Distributed locking
* Asynchronous notifications
* Request validation
* Correlation ID tracking
* Structured logging
* Comprehensive error handling

---

# 🔒 Distributed Booking Protection

Booking systems must handle situations where multiple customers attempt to reserve the same resource simultaneously.

This project uses **Redis and Redlock-based distributed locking** to prevent race conditions during booking operations.

Distributed locks ensure that critical booking resources cannot be modified concurrently in ways that could produce inconsistent reservation data.

This architecture is particularly important when multiple instances of the Booking Service are running simultaneously.

---

# 🔁 Idempotent Reservations

Booking confirmation operations support idempotency keys.

Each booking can be associated with a unique identifier that prevents accidental duplicate operations caused by:

* Network retries
* Repeated requests
* Client-side resubmissions
* Service communication failures

This makes reservation processing more reliable and closer to the architecture expected from production booking systems.

---

# ⭐ Reviews & Ratings

The Review Service is implemented in Go and manages guest feedback associated with properties and completed bookings.

### Features

* Create reviews
* Retrieve all reviews
* Retrieve individual reviews
* Rating validation
* User association
* Property association
* Booking association
* Structured API responses
* Data integrity validation

A review can be associated with a specific:

* User
* Property
* Booking
* Rating
* Comment

This creates a foundation for verified-stay review systems.

---

# 📧 Notification Service

The Notification Service handles asynchronous customer communication.

### Features

* Email notifications
* Welcome emails
* Booking-confirmation emails
* Template-based messaging
* Background processing
* Redis queues
* Queue workers
* Email-job processing
* Error handling
* Structured logging
* Health monitoring

Emails are generated using reusable Handlebars templates and delivered through Nodemailer.

---

# ⚡ Asynchronous Processing

Operations that do not need to block the main HTTP request are processed asynchronously.

The project uses:

* **Redis**
* **BullMQ**
* **Background workers**

for queue-driven processing.

For example, a booking can be completed immediately while its confirmation email is placed into a background queue.

This architecture improves:

* API responsiveness
* Scalability
* Reliability
* Failure recovery
* Service isolation

---

# 🔄 Microservices Communication

Services communicate using two primary mechanisms.

## Synchronous Communication

REST APIs are used when one service requires an immediate response from another service.

## Asynchronous Communication

Redis and BullMQ queues are used for operations that can be processed independently, such as email notifications.

Correlation IDs are propagated through requests to make it easier to trace operations across multiple services.

---

# 🏗️ Architecture

```text
                        ┌──────────────────────┐
                        │       Client         │
                        └──────────┬───────────┘
                                   │
                                   ▼
                        ┌──────────────────────┐
                        │   Auth Service       │
                        │       Go             │
                        └──────┬────────┬──────┘
                               │        │
                  ┌────────────┘        └────────────┐
                  ▼                                  ▼
        ┌──────────────────┐              ┌──────────────────┐
        │  Hotel Service   │              │ Booking Service  │
        │ Node/TypeScript  │              │ Node/TypeScript  │
        └────────┬─────────┘              └────────┬─────────┘
                 │                                 │
                 │                                 ▼
                 │                       ┌──────────────────┐
                 │                       │      Redis       │
                 │                       │ Queues / Locks   │
                 │                       └────────┬─────────┘
                 │                                │
                 ▼                                ▼
        ┌──────────────────┐             ┌────────────────────┐
        │  Review Service  │             │Notification Service│
        │        Go        │             │  Node/TypeScript   │
        └──────────────────┘             └────────────────────┘

                         ┌─────────────────┐
                         │      MySQL      │
                         │ Persistent Data │
                         └─────────────────┘
```

---

# 🧩 Microservices

## AuthInGo

**Language:** Go

Responsible for:

* Authentication
* User accounts
* JWT tokens
* OTP verification
* Authorization
* Roles
* Permissions
* Security middleware
* Service proxying

---

## BookingService

**Language:** TypeScript
**Framework:** Express.js

Responsible for:

* Reservation creation
* Reservation confirmation
* Reservation cancellation
* Booking status
* Idempotency
* Database transactions
* Redis locking
* Background jobs

---

## HotelService

**Language:** TypeScript
**Framework:** Express.js

Responsible for:

* Properties
* Hotels
* Rooms
* Room categories
* Availability
* Pricing
* Ratings
* Property search
* Property management

---

## NotificationService

**Language:** TypeScript
**Framework:** Express.js

Responsible for:

* Email delivery
* Email templates
* Booking notifications
* Welcome emails
* Background jobs
* Queue processing

---

## ReviewService

**Language:** Go

Responsible for:

* Guest reviews
* Property ratings
* Booking-linked reviews
* Review retrieval
* Review validation

---

# 🛠️ Technology Stack

## Languages

* Go
* TypeScript
* JavaScript

## Backend

* Node.js
* Express.js
* Go Chi Router

## Databases

* MySQL
* MongoDB

## ORM & Data Access

* Prisma
* Sequelize
* Repository Pattern

## Authentication & Security

* JWT
* bcrypt
* OTP verification
* Role-Based Access Control
* Rate limiting
* Input validation

## Validation

* Zod
* Go Validator

## Cache & Distributed Systems

* Redis
* IORedis
* Redlock

## Background Processing

* BullMQ
* Redis queues
* Worker processes

## Notifications

* Nodemailer
* Handlebars
* SMTP

## Logging & Observability

* Winston
* MongoDB logging
* Daily rotating logs
* Correlation IDs
* Health-check endpoints

---

# 🗄️ Data Layer

The project uses MySQL as its primary relational database.

Different services use dedicated data-access technologies appropriate to their implementation:

* **Prisma** for booking-related persistence
* **Sequelize** for hotel/property data
* Native Go database access for Go services
* **MongoDB** for selected logging functionality

Database migrations provide version-controlled schema changes.

Transactions are used for operations where data consistency is critical.

---

# 🚀 Scalability

The microservice architecture allows different areas of the platform to scale independently.

For example:

* Booking workers can scale during periods of high reservation traffic.
* Notification workers can scale when large volumes of emails are queued.
* Property-search services can be replicated independently.
* Authentication services can be scaled according to login traffic.

Redis-backed queues and distributed locking help coordinate work across multiple application instances.

---

# 🔍 Reliability & Observability

The platform implements several features designed to improve production reliability:

* Health-check endpoints
* Structured logging
* Correlation IDs
* Request validation
* Centralized error handling
* Database transactions
* Idempotent operations
* Distributed locking
* Queue-based processing
* Soft deletion
* Retry-friendly architecture

These features make the project more representative of a real distributed backend rather than a basic CRUD application.

---

# 🌍 GuestReady-Inspired Experience

The project takes inspiration from platforms such as GuestReady, where accommodation booking is combined with the operational systems required to manage hospitality properties.

A production version of the platform can support workflows such as:

### For Guests

* Search accommodations
* Check availability
* Make reservations
* Manage bookings
* Receive booking notifications
* Review completed stays

### For Property Managers

* Add properties
* Manage rooms
* Set room availability
* Manage pricing
* Track bookings
* Review guest feedback

### For Administrators

* Manage users
* Manage permissions
* Manage roles
* Monitor properties
* Monitor bookings
* Moderate reviews
* Manage platform operations

---

# 🎯 Project Purpose

The goal of this project is to demonstrate the architecture behind a **production-style hospitality and accommodation booking platform**.

Rather than concentrating only on UI replication, the project focuses heavily on backend engineering and distributed-system concepts required by real booking platforms.

It demonstrates practical experience with:

* Microservices architecture
* Distributed systems
* REST API design
* Go backend development
* Node.js backend development
* TypeScript
* Authentication
* Authorization
* OTP verification
* Role-based access control
* Database architecture
* Redis
* Distributed locking
* Message queues
* Background processing
* Idempotency
* Database transactions
* Property management
* Booking management
* Review systems
* Email infrastructure
* Logging
* Observability
* Error handling
* API validation
* Scalable service design

The result is a modular hospitality backend capable of serving as the foundation for a large-scale property-management and accommodation-booking application.
