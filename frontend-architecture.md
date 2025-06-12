# Frontend Architecture

## 🧱 System Architecture

### Overview
The frontend for the Telecom Mission Control SaaS platform is designed as a modular, scalable, and accessible system using a **micro-frontend (MFE)** approach. The application supports both web and mobile users, works globally, and provides robust offline capabilities.

### Container App (Shell)
- Handles global layout, navigation, and routing.
- Orchestrates and loads MFEs dynamically.
- Provides shared services: authentication, theme, accessibility, and offline support.

### Micro Frontend Apps (MFEs)
- **Dashboard:** Real-time monitoring, charts, and security health.
- **Policy Manager:** Policy creation, editing, and enforcement.
- **Device List:** Device discovery, status, and management.
- **User Roles:** User onboarding, role assignment, and management.

Each MFE is independently deployable and communicates via shared events and APIs.

### Offline Support
- **IndexedDB & Local Storage:** Persist user actions, device data, and policy changes for offline access.
- **Service Workers:** Cache static assets and API responses; queue offline actions for background sync and synchronization when online.

### API Adapter Layer
- Unified interface for GraphQL and RESTful backend services.
- Handles authentication tokens, error handling, and data normalization.
- Supports both online and queued (offline) API requests.

### Theme & Accessibility Provider
- Centralized theming (light/dark, high contrast).
- Accessibility features: ARIA, keyboard navigation, font scaling.
- Ensures WCAG 2.1 AA compliance across all MFEs.

---

## 🔄 Data Flow Models

### Device Discovery
- **Trigger:** Cell tower scan detects new device.
- **Steps:**
  1. Tower scan event triggers device discovery in Device List MFE.
  2. Device info is stored in IndexedDB (offline-first).
  3. If online, device info is synced to backend via API Adapter.
  4. If offline, sync is queued and retried when connection is restored.

### User Login + Role Assignment
- **Auth Provider:** Auth0 or Firebase Auth.
- **Steps:**
  1. User logs in via OAuth.
  2. JWT is issued and stored securely.
  3. API Adapter uses JWT for all API requests.
  4. User role is fetched and determines which UI components are rendered.

### Policy Enforcement UI
- **Steps:**
  1. Admin selects a user role.
  2. Action matrix/permissions grid is displayed.
  3. Admin modifies allowed actions.
  4. Changes are saved to backend via API Adapter.
  5. Updates are reflected in UI and synchronized for offline support.

### Dashboard Monitoring
- **Steps:**
  1. Dashboard MFE subscribes to WebSocket channels per device group.
  2. Real-time telemetry and alerts are pushed from backend.
  3. Charts and state update live in the UI.
  4. If offline, latest cached data is shown with a warning.

---


