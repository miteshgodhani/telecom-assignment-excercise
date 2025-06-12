# **Frontend Architecture**

## 🧱 **System Architecture**

### **Overview**
The frontend for the Telecom Mission Control SaaS platform is designed as a **modular, scalable, and accessible system** using a **micro-frontend (MFE)** approach. The application supports both **web and mobile users**, works globally, and provides robust **offline capabilities**. It ensures high-assurance functionality, accessibility compliance, and seamless integration with backend services.

---

### **Container App (Shell)**
- **Responsibilities**:
  - Handles **global layout**, navigation, and routing.
  - Orchestrates and dynamically loads **MFEs**.
  - Provides shared services:
    - **Authentication** (e.g., OAuth, JWT management).
    - **Theme management** (light/dark mode, high contrast).
    - **Accessibility** (ARIA roles, keyboard navigation, font scaling).
    - **Offline support** (service workers, IndexedDB).
  - Acts as the **single entry point** for the application.

- **Features**:
  - **Scalability**: Supports independent deployment of MFEs.
  - **Global State Management**: Uses a shared state provider (e.g., React Context or Redux) for cross-MFE communication.
  - **Performance Optimization**: Lazy loading of MFEs and shared libraries.

---

### **Micro Frontend Apps (MFEs)**
Each MFE is independently deployable, scalable, and communicates via shared events and APIs. MFEs are designed to handle specific features of the platform.

#### **Dashboard MFE**
- **Purpose**: Real-time monitoring, charts, and security health.
- **Features**:
  - Subscribes to **WebSocket channels** for real-time telemetry and alerts.
  - Displays **charts**, **graphs**, and **security health metrics**.
  - Offline mode: Shows cached data with warnings when disconnected.

#### **Policy Manager MFE**
- **Purpose**: Policy creation, editing, and enforcement.
- **Features**:
  - Allows admins to define and enforce policies for user roles.
  - Displays an **action matrix/permissions grid** for policy management.
  - Offline mode: Saves policy changes locally and syncs them when online.

#### **Device List MFE**
- **Purpose**: Device discovery, status, and management.
- **Features**:
  - Displays a list of enterprise devices detected by cell towers.
  - Allows admins to manage device statuses (active/inactive).
  - Offline mode: Stores device data locally and syncs it with the backend when online.

#### **User Roles MFE**
- **Purpose**: User onboarding, role assignment, and management.
- **Features**:
  - Allows admins to assign roles to users and manage permissions.
  - Fetches user roles dynamically based on enterprise policies.
  - Offline mode: Role assignments are queued for sync when online.

---

### **Offline Support**
The frontend is designed to work seamlessly in both **online and offline modes** using **PWA principles**.

#### **IndexedDB & Local Storage**
- **Purpose**: Persist structured data for offline access.
- **Usage**:
  - **IndexedDB**: Stores user actions, device data, and policy changes.
  - **Local Storage**: Stores lightweight data such as user preferences and session tokens.

#### **Service Workers**
- **Purpose**: Enable offline capabilities and background synchronization.
- **Features**:
  - **Static Asset Caching**: Cache JavaScript, CSS, and images for offline access.
  - **API Response Caching**: Cache frequently accessed API responses (e.g., dashboard metrics, device data).
  - **Background Sync**: Queue offline actions (e.g., policy updates, device registrations) and synchronize them with the backend when online.

#### **ReactJS PWA Implementation**
- **Setup**:
  - Service workers are registered using Workbox.
  - Offline actions are queued using Workbox Background Sync.
- **Behavior**:
  - When offline, user actions are queued and stored locally.
  - When online, queued actions are automatically retried and synchronized with the backend.

---

### **API Adapter Layer**
The **API Adapter Layer** provides a unified interface for backend services and ensures seamless communication between the frontend and backend.

#### **Features**:
- **Unified Interface**:
  - Supports both **GraphQL** and **RESTful APIs**.
  - Abstracts API differences for consistent frontend integration.
- **Authentication**:
  - Manages **JWT tokens** for secure API requests.
  - Automatically refreshes tokens when expired.
- **Error Handling**:
  - Centralized error handling for API failures.
  - Displays user-friendly error messages in the UI.
- **Data Normalization**:
  - Normalizes API responses for consistent data structures across MFEs.
- **Offline Support**:
  - Queues API requests when offline and retries them when online.

---

### **Theme & Accessibility Provider**
The frontend ensures **WCAG 2.1 AA compliance** and provides a centralized theming and accessibility system.

#### **Features**:
- **Theming**:
  - Supports **light/dark mode** and **high contrast themes**.
  - Dynamically adjusts UI components based on user preferences.
- **Accessibility**:
  - Implements **ARIA roles** for screen readers.
  - Supports **keyboard navigation** and **font scaling**.
  - Ensures all MFEs are accessible to users with disabilities.

---

## 🔄 **Data Flow Models**

### **Device Discovery**
- **Trigger**: Cell tower scan detects a new device.
- **Steps**:
  1. Tower scan event triggers device discovery in the **Device List MFE**.
  2. Device info is stored in **IndexedDB** for offline-first access.
  3. If online, device info is synced to the backend via the **API Adapter**.
  4. If offline, sync is queued and retried when the connection is restored.

---

### **User Login + Role Assignment**
- **Trigger**: User logs in via OAuth.
- **Steps**:
  1. User logs in using **Auth0** or **Firebase Auth**.
  2. A **JWT token** is issued and stored securely.
  3. The **API Adapter** uses the JWT for all API requests.
  4. User role is fetched and determines which UI components are rendered.

---

### **Policy Enforcement UI**
- **Trigger**: Admin modifies policy rules for a user role.
- **Steps**:
  1. Admin selects a user role in the **Policy Manager MFE**.
  2. An **action matrix/permissions grid** is displayed.
  3. Admin modifies allowed actions for the role.
  4. Changes are saved to the backend via the **API Adapter**.
  5. Updates are reflected in the UI and synchronized for offline support.

---

### **Dashboard Monitoring**
- **Trigger**: Real-time telemetry and alerts are pushed from the backend.
- **Steps**:
  1. The **Dashboard MFE** subscribes to **WebSocket channels** for device groups.
  2. Real-time telemetry and alerts are pushed from the backend.
  3. Charts and state are updated live in the UI.
  4. If offline, the latest cached data is shown with a warning.

---

## **Improvements Added**
1. **Scalability**:
   - Added lazy loading for MFEs and shared libraries to optimize performance.
   - Enhanced modularity for independent deployment of MFEs.
2. **Accessibility**:
   - Ensured WCAG 2.1 AA compliance across all MFEs.
   - Added ARIA roles, keyboard navigation, and font scaling.
3. **Offline Support**:
   - Integrated Workbox Background Sync for queuing offline actions.
   - Improved caching strategies for static assets and API responses.
4. **API Integration**:
   - Unified API Adapter for GraphQL and RESTful services.
   - Centralized error handling and data normalization.