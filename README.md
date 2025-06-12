# Telecom Mission Control SaaS Platform - Frontend Architecture

This project is a frontend architecture plan for a telecom mission control SaaS platform. It addresses real-time device monitoring, role-based access enforcement, and multi-tenant micro-frontend (MFE) architecture to support a11y-friendly, responsive, offline-capable, and scalable experiences across web and mobile devices.,with offline/online capabilities.

## Features

- Create a globally accessible, responsive frontend system.
- Handle both online and offline modes.
- Provide real-time dashboards and user policy enforcement.
- Support multiple carriers, OS/device types, and user roles.

## Architecture Highlights

- Micro Frontend Architecture (MFE) using Module SingleSpa.
- React + TypeScript UI components with PWA support.
- Service Workers for offline caching.
- Accessibility (a11y) best practices (WCAG 2.1).
- Real-time updates using WebSocket for dashboards.
- RESTful APIs + GraphQL hybrid model for efficient data querying.

See [frontend-architecture.md](./frontend-architecture.md) for a detailed overview.

## Technologies Used

- React, TypeScript, Tailwind CSS, React Native(for mobile)
- Redux Toolkit, RTK Query, React Query
- Webpack
- Lerna - Monorepo tool
- GraphQL (Apollo Client)
- Service Workers, IndexedDB
- Storybook for UI development

## API Models

API model documentation is in [api/data-model](./api/data-flow-api-model.md).

## Data Flows & Diagrams

All system diagrams are in [diagrams/](./diagrams/).

## Frontend web project scafolding

All system diagrams are in [diagrams/](./diagrams/).