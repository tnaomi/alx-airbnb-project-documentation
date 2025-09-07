# 📄 Backend Feature Requirement Specifications

This document captures the technical and functional requirements for core backend features of the **Airbnb Clone**.

---

## 1. 🔐 User Authentication

### **Description**
Handles secure registration, login, and authentication for users (guests, hosts, admins).

### **API Endpoints**
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/profile` (protected)

### **Input / Output**
**Registration (POST /api/auth/register)**  
- Input (JSON):
  ```json
  {
    "email": "user@example.com",
    "password": "SecurePass123",
    "role": "guest"
  }
- Output:

```bash
{
  "message": "User registered successfully",
  "userId": "uuid"
}
```

**Login (POST /api/auth/login)**
- INput (JSON):
```json
{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

- Output 

```bash
{
  "token": "jwt_token_here",
  "expiresIn": 3600
}
```

### Validation Rules

Email must be unique and valid format.

Password: minimum 8 chars, must include letters and numbers.

Role restricted to guest or host.

### Performance Criteria

Authentication response time < 300ms under normal load.

Support 1000 concurrent logins without degradation.

## 2. 🏡 Property Management

### Description

Hosts can create, edit, delete, and manage property listings.

### API Endpoints

POST `/api/properties`

GET `/api/properties`

GET `/api/properties/:id`

PUT `/api/properties/:id`

DELETE `/api/properties/:id`

### Input / Output

- Create Listing (`POST /api/properties`)

- - Input:

```json
{
  "title": "Beachfront Villa",
  "description": "Beautiful villa with ocean view",
  "location": "Malawi, Cape Maclear",
  "price": 150,
  "amenities": ["WiFi", "Pool", "Air Conditioning"],
  "availability": {
    "start": "2025-09-10",
    "end": "2025-12-31"
  }
}
```
- - Output
```json
{
  "message": "Property created successfully",
  "propertyId": "uuid"
}
```

- Get Listings (`GET /api/properties`)

- - Output:

```json
[
  {
    "id": "uuid",
    "title": "Beachfront Villa",
    "location": "Cape Maclear",
    "price": 150,
    "amenities": ["WiFi", "Pool"]
  }
]
```

### Validation Rules

- *Title* max 100 characters.

- *Price* must be > 0.

- Availability dates must not overlap with existing bookings.

### Performance Criteria

- Fetch listings with filters in < 500ms.

- Paginate results (default: 10 per page, max: 50).

## 3. 📅 Booking System

### Description

- Allows guests to book properties, manage cancellations, and track booking status.

### API Endpoints

- POST `/api/bookings`

- GET `/api/bookings/:id`

- DELETE `/api/bookings/:id/cancel`

- GET `/api/bookings/user/:userId`

### Input / Output

#### Create Booking (POST `/api/bookings`)

- Input: 
```json
{
  "propertyId": "uuid",
  "guestId": "uuid",
  "checkIn": "2025-09-15",
  "checkOut": "2025-09-20",
  "guests": 2
}
```
- Output:

``` json
{
  "message": "Booking created successfully",
  "bookingId": "uuid",
  "status": "pending"
}
```

#### Cancel Booking (DELETE `/api/bookings/:id/cancel`)

- Output
```json
{
  "message": "Booking cancelled successfully",
  "status": "cancelled"
}
```

### Validation Rules

- Dates must not overlap with existing confirmed bookings.

- Check-out date must be later than check-in date.

- Maximum number of guests must not exceed property capacity.

### Performance Criteria

- Booking creation < 400ms response time.

- System should prevent double-bookings through transactional integrity.

# 📊 Requirements Traceability Matrix (RTM)

The RTM ensures that all technical and functional requirements are mapped to user stories and corresponding test cases.

---

## 1. User Authentication

| Requirement ID | Requirement Description | User Story | API Endpoint(s) | Test Cases |
|----------------|--------------------------|------------|-----------------|------------|
| AUTH-001 | Allow user registration with email & password | As a user, I want to register an account so I can log in and use the platform. | `POST /api/auth/register` | Verify registration with valid inputs; verify rejection with invalid email/password |
| AUTH-002 | Enable secure login with JWT-based authentication | As a user, I want to log in so I can access my dashboard. | `POST /api/auth/login` | Verify login with valid credentials; verify failure with wrong password |
| AUTH-003 | Password reset functionality | As a user, I want to reset my password if I forget it. | `POST /api/auth/reset-password` | Verify reset with valid email; verify rejection for unregistered email |
| AUTH-004 | Enforce strong password policy | As a user, I want the system to enforce strong passwords for better security. | `POST /api/auth/register` | Verify rejection of weak passwords; verify acceptance of strong ones |

---

## 2. Property Management

| Requirement ID | Requirement Description | User Story | API Endpoint(s) | Test Cases |
|----------------|--------------------------|------------|-----------------|------------|
| PROP-001 | Allow users to add property listings | As a host, I want to list my property so that users can book it. | `POST /api/properties` | Verify creation with valid data; reject with missing title/price |
| PROP-002 | Retrieve property details | As a user, I want to view property details so I can decide on booking. | `GET /api/properties/{id}` | Verify retrieval of existing property; verify 404 for invalid ID |
| PROP-003 | Update property information | As a host, I want to update my property details to keep them accurate. | `PUT /api/properties/{id}` | Verify update with valid fields; reject unauthorized update |
| PROP-004 | Delete property listing | As a host, I want to delete my property if it’s no longer available. | `DELETE /api/properties/{id}` | Verify deletion of property; verify unauthorized access is denied |

---

## 3. Booking System

| Requirement ID | Requirement Description | User Story | API Endpoint(s) | Test Cases |
|----------------|--------------------------|------------|-----------------|------------|
| BOOK-001 | Allow users to book available properties | As a user, I want to book a property for specific dates. | `POST /api/bookings` | Verify booking with valid dates; reject booking for unavailable property |
| BOOK-002 | Prevent double booking | As a system, I must ensure a property can’t be booked twice for the same dates. | `POST /api/bookings` | Attempt to double-book same property; verify rejection |
| BOOK-003 | Allow cancellation of bookings | As a user, I want to cancel my booking if plans change. | `DELETE /api/bookings/{id}` | Verify successful cancellation; verify error for invalid booking ID |
| BOOK-004 | Retrieve booking details | As a user, I want to view my booking details for confirmation. | `GET /api/bookings/{id}` | Verify retrieval of valid booking; verify error for unauthorized access |

---

## 4. Payment Integration

| Requirement ID | Requirement Description | User Story | API Endpoint(s) | Test Cases |
|----------------|--------------------------|------------|-----------------|------------|
| PAY-001 | Allow secure payment for bookings | As a user, I want to pay securely so that my booking is confirmed. | `POST /api/payments/checkout` | Verify payment success with valid details; reject with insufficient funds |
| PAY-002 | Support multiple payment methods | As a user, I want to pay with different methods (card, PayPal, mobile money). | `POST /api/payments/checkout` | Verify card payment, PayPal, and mobile money flows |
| PAY-003 | Confirm payment after gateway response | As a system, I must confirm payment status with the gateway before finalizing booking. | `POST /api/payments/confirm` | Simulate gateway success/failure; verify booking status update |
| PAY-004 | Process refunds | As a user, I want a refund if I cancel before the deadline. | `POST /api/payments/refund` | Verify refund with valid booking; reject refund request after policy deadline |
| PAY-005 | Log all transactions for auditing | As an admin, I want to see all transactions for compliance. | DB logging + `GET /api/payments/history` | Verify transaction logging after successful payment |

---

## Summary
This RTM establishes clear traceability:
- **Requirements** ↔ **User Stories** ↔ **API Endpoints** ↔ **Test Cases**  
ensuring **coverage, accountability, and audit-readiness** for **Authentication, Property Management, Booking, and Payment** modules.
