# Data Flow Models and API Design

This document provides a detailed breakdown of **data flow models** for each feature, along with the **API model** and guidelines for designing a **performant API** for the system.

---

## **1. Data Flow Models for Each Feature**

### **Feature 1: Real-Time Dashboard**
**Purpose**: Display real-time metrics and security health updates, including **which user used which app**, **at which tower**, and **violated which policy**.

**Data Flow**:
1. **Frontend**:
   - User logs in and navigates to the dashboard.
   - The frontend sends an API request to fetch detailed app usage and policy violations.
   - A WebSocket connection is established for real-time updates.
2. **Backend**:
   - The `APP_USAGE` table is queried to fetch app usage logs, including `user_id`, `app_id`, and `cell_tower_id`.
   - The `POLICY_RULE` table is queried to check for violations based on the user's role and actions.
   - Real-time updates are pushed to the frontend via WebSocket.
3. **External Systems**:
   - Data from `CELL_TOWER` and `TOWER_CARRIER` is ingested into the backend.

**Entities Involved**:
- `USER`, `DEVICE`, `APP_USAGE`, `APP`, `CELL_TOWER`, `POLICY_RULE`

---

### **Feature 2: Device Auto-Discovery**
**Purpose**: Automatically detect and onboard new enterprise devices.

**Data Flow**:
1. **Frontend**:
   - Periodically polls the backend or listens for WebSocket updates for newly discovered devices.
   - Displays the list of discovered devices in the UI.
2. **Backend**:
   - The `DEVICE` table is updated with new devices detected by `CELL_TOWER`.
   - The `OS` table is referenced to associate the operating system with the device.
   - Sends updates to the frontend via WebSocket or API.
3. **External Systems**:
   - `CELL_TOWER` sends device discovery data to the backend.

**Entities Involved**:
- `DEVICE`, `OS`, `CELL_TOWER`, `LOCATION`

---

### **Feature 3: Policy Enforcement**
**Purpose**: Enforce role-based policies to allow or deny actions.

**Data Flow**:
1. **Frontend**:
   - Fetches policies from the backend based on the user's role.
   - Validates user actions against the policies.
2. **Backend**:
   - The `POLICY_RULE` table is queried to fetch rules for the user's role and app.
   - The `ACTION` table is referenced to determine allowed or denied actions.
3. **External Systems**:
   - Policies may be enforced at the `CELL_TOWER` level for edge security.

**Entities Involved**:
- `USER`, `ROLE`, `POLICY`, `POLICY_RULE`, `APP`, `ACTION`

---

### **Feature 4: Auto-Remedial Actions**
**Purpose**: Automatically take corrective actions based on alerts.

**Data Flow**:
1. **Frontend**:
   - Displays notifications about auto-remedial actions taken by the system.
2. **Backend**:
   - The `PLAN` table is checked to verify if the enterprise has subscribed to auto-remedial actions.
   - The `POLICY_RULE` table is queried to determine the appropriate action.
   - The `CELL_TOWER` is updated to enforce the action.
3. **External Systems**:
   - `CELL_TOWER` executes the remedial actions (e.g., blocking a device or app).

**Entities Involved**:
- `PLAN`, `ENTERPRISE`, `POLICY_RULE`, `CELL_TOWER`, `ACTION`

---

## **2. API Model**

### **API Endpoints**

#### **Authentication**
- `POST /auth/login`: Authenticate a user and return a token.
- `POST /auth/refresh`: Refresh the authentication token.

#### **Dashboard**
- `GET /dashboard/metrics`: Fetch detailed app usage and policy violations.

#### **Devices**
- `GET /devices`: Fetch registered devices.
- `POST /devices`: Register a new device.
- `GET /devices/discovered`: Fetch auto-discovered devices.

#### **Policies**
- `GET /policies`: Fetch policies for the user or role.
- `POST /policies`: Create or update a policy.

#### **Policy Rules**
- `GET /policy-rules`: Fetch policy rules for a specific policy.
- `POST /policy-rules`: Add or update a policy rule.

#### **Roles**
- `POST /roles/policies/assign`: Assign a policy to a role.
- `POST /users/roles/assign`: Assign a role to a user.
- `GET /roles/policies?role`: Fetch policies assigned to a role.
- `GET /users/roles?user`: Fetch roles assigned to a user.
- `DELETE /roles/{role_id}/policies/{policy_id}`: Remove a policy from a role.
- `DELETE /users/{user_id}/roles/{role_id}`: Remove a role from a user.


#### **Carriers and Towers**
- `GET /carriers`: Fetch supported carriers.
- `GET /towers`: Fetch tower information.

#### **Real-Time Alerts**
- WebSocket endpoint: `/ws/alerts`: Subscribe to real-time alerts.

---

### **API Request/Response Examples**

Here are additional **API Request/Response Examples** for various endpoints, covering more scenarios and use cases:

---

### **1. Fetch Registered Devices**
**Request**:
```http
GET /devices HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "devices": [
    {
      "id": "device123",
      "user_id": "user123",
      "user_name": "John Doe",
      "model": "iPhone 13",
      "os": "iOS",
      "status": "active",
      "last_seen": "2025-06-12T09:30:00Z"
    },
    {
      "id": "device124",
      "user_id": "user124",
      "user_name": "Jane Smith",
      "model": "Samsung Galaxy S21",
      "os": "Android",
      "status": "inactive",
      "last_seen": "2025-06-11T15:45:00Z"
    }
  ]
}
```

---

### **2. Register a New Device**
**Request**:
```http
POST /devices HTTP/1.1
Authorization: Bearer <token>
Content-Type: application/json

{
  "user_id": "user123",
  "model": "Google Pixel 6",
  "os": "Android"
}
```

**Response**:
```json
{
  "device_id": "device125",
  "user_id": "user123",
  "model": "Google Pixel 6",
  "os": "Android",
  "status": "registered",
  "registered_at": "2025-06-12T10:00:00Z"
}
```

---

### **3. Fetch Policies for a User**
**Request**:
```http
GET /policies?user_id=user123 HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "user_id": "user123",
  "policies": [
    {
      "policy_id": "policy123",
      "name": "Admin Policy",
      "rules": [
        {
          "app_name": "Slack",
          "action": "Upload File",
          "permission": "deny"
        },
        {
          "app_name": "Zoom",
          "action": "Start Meeting",
          "permission": "allow"
        }
      ]
    }
  ]
}
```

---

### **4. Add or Update a Policy Rule**
**Request**:
```http
POST /policy-rules HTTP/1.1
Authorization: Bearer <token>
Content-Type: application/json

{
  "policy_id": "policy123",
  "role_id": "role456",
  "app_id": "app123",
  "action_id": "action789",
  "permission": "deny"
}
```

**Response**:
```json
{
  "policy_rule_id": "rule123",
  "policy_id": "policy123",
  "role_id": "role456",
  "app_id": "app123",
  "action_id": "action789",
  "permission": "deny",
  "status": "updated",
  "updated_at": "2025-06-12T10:15:00Z"
}
```

---

### **5. Fetch Tower Information**
**Request**:
```http
GET /towers HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "towers": [
    {
      "id": "tower123",
      "location": "New York City",
      "carriers": ["AT&T", "Verizon"],
      "status": "active"
    },
    {
      "id": "tower124",
      "location": "San Francisco",
      "carriers": ["T-Mobile"],
      "status": "inactive"
    }
  ]
}
```

---

### **6. Fetch Real-Time Alerts**
**Request**:
```http
GET /dashboard/alerts HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "alerts": [
    {
      "alert_id": "alert123",
      "user_id": "user123",
      "user_name": "John Doe",
      "app_name": "Slack",
      "action": "Upload File",
      "policy_name": "No File Uploads",
      "severity": "high",
      "timestamp": "2025-06-12T10:00:00Z"
    },
    {
      "alert_id": "alert124",
      "user_id": "user124",
      "user_name": "Jane Smith",
      "app_name": "Zoom",
      "action": "Start Meeting",
      "policy_name": "Restricted Meeting Access",
      "severity": "medium",
      "timestamp": "2025-06-12T10:05:00Z"
    }
  ]
}
```

---

### **7. Trigger Auto-Remedial Action**
**Request**:
```http
POST /remedial-actions HTTP/1.1
Authorization: Bearer <token>
Content-Type: application/json

{
  "alert_id": "alert123",
  "action": "block_device"
}
```

**Response**:
```json
{
  "action_id": "action123",
  "alert_id": "alert123",
  "action_name": "Block Device",
  "status": "success",
  "executed_at": "2025-06-12T10:15:00Z"
}
```

---

### **8. Fetch Role-Based Policies**
**Request**:
```http
GET /policies?role_id=role123 HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "role_id": "role123",
  "role_name": "Admin",
  "policies": [
    {
      "policy_id": "policy123",
      "name": "Admin Policy",
      "rules": [
        {
          "app_name": "Slack",
          "action": "Upload File",
          "permission": "deny"
        },
        {
          "app_name": "Zoom",
          "action": "Start Meeting",
          "permission": "allow"
        }
      ]
    }
  ]
}
```

---

### **9. Fetch App Usage Logs**
**Request**:
```http
GET /app-usage?user_id=user123 HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "user_id": "user123",
  "user_name": "John Doe",
  "app_usage": [
    {
      "app_id": "app123",
      "app_name": "Slack",
      "cell_tower_id": "tower456",
      "cell_tower_location": "New York City",
      "timestamp": "2025-06-12T10:00:00Z"
    },
    {
      "app_id": "app124",
      "app_name": "Zoom",
      "cell_tower_id": "tower789",
      "cell_tower_location": "San Francisco",
      "timestamp": "2025-06-12T10:05:00Z"
    }
  ]
}
```

---

### **10. Fetch Carrier Information**
**Request**:
```http
GET /carriers HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "carriers": [
    {
      "id": "carrier123",
      "name": "AT&T",
      "supported_towers": ["tower123", "tower124"]
    },
    {
      "id": "carrier124",
      "name": "Verizon",
      "supported_towers": ["tower123"]
    }
  ]
}
```



---

### **11. Fetch Auto-Discovered Devices**
**Request**:
```http
GET /devices/discovered HTTP/1.1
Authorization: Bearer <token>
```

**Response**:
```json
{
  "devices": [
    {
      "id": "device123",
      "model": "iPhone 13",
      "os": "iOS",
      "status": "discovered"
    },
    {
      "id": "device124",
      "model": "Samsung Galaxy S21",
      "os": "Android",
      "status": "discovered"
    }
  ]
}
```

---

## **3. Designing a Performant API**

### **Key Strategies**

1. **Optimize Database Queries**:
   - Use indexes for frequently queried fields (e.g., `user_id`, `device_id`, `policy_id`).
   - Avoid N+1 query problems by using joins or batch queries.

2. **Caching**:
   - Cache frequently accessed data (e.g., policies, app usage) using tools like **Redis**.
   - Use `ETag` headers for client-side caching.

3. **Asynchronous Processing**:
   - Offload heavy tasks (e.g., auto-remedial actions) to background workers using a message queue (e.g., RabbitMQ, Kafka).

4. **WebSocket for Real-Time Updates**:
   - Use WebSocket for pushing real-time alerts instead of polling.
   - Ensure WebSocket connections are load-balanced and scalable.

5. **Rate Limiting**:
   - Implement rate limiting to prevent abuse.
   - Example: Allow 100 requests per minute per user.

6. **Pagination**:
   - For large datasets (e.g., devices, app usage), implement pagination using query parameters:
     - Example: `GET /devices?page=1&limit=20`.

7. **API Gateway**:
   - Use an API gateway (e.g., AWS API Gateway, Kong) for centralized authentication, rate limiting, and monitoring.