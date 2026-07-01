# Netiks Store Project Documentation

## Overview

Netiks Store is a multi-vendor e-commerce platform with a customer storefront, seller onboarding, product publishing, checkout, and supporting backend services.

This document gives a simple view of the product, the main services, and how the application is structured. For the fastest local setup, use [INTERN_QUICKSTART_GUIDE.md](/Users/woron/Documents/netiks-store/docs/INTERN_QUICKSTART_GUIDE.md).

## What The Application Does

- Customers can browse products, open product pages, and place orders.
- Sellers can register, sign in, create a store, publish products, and track orders.
- The platform separates frontend, gateway, identity, catalog, vendor, media, and admin responsibilities into individual services.

## Main User Flows

### Customer flow

- Open the storefront
- Browse the market
- View a product
- Continue to checkout
- Place an order

### Seller flow

- Register a seller account
- Sign in
- Create a store
- Add categories
- Publish products
- Track sales and orders

## Architecture At A Glance

### Frontend

- `apps/web`: Next.js application for customers and sellers

### Backend services

- `services/gateway`: central API entry point used by the frontend
- `services/identity-service`: registration, login, and user identity
- `services/vendor-service`: seller and store information
- `services/catalog-service`: categories, products, checkout, and orders
- `services/media-service`: image uploads and media handling
- `services/admin-service`: admin-facing capabilities

### Supporting infrastructure

- `postgres`: primary database
- `redis`: support service for shared runtime needs
- `docker-compose.yml`: local orchestration for the full stack

## Technology Stack

- Frontend: Next.js and TypeScript
- Backend: FastAPI and Python
- Database: PostgreSQL
- Container runtime: Docker Compose

## Local Runtime Model

The project is designed to run as a multi-service stack locally. Each service starts inside Docker, which makes local execution closer to a production-style deployment than a single-process app.

This gives interns a simple pattern to follow:

- start the full stack
- verify the containers
- open the frontend
- confirm API health
- reload marketplace data when needed

## Key Repository Paths

```text
netiks-store/
  apps/
    web/
  services/
    gateway/
    identity-service/
    vendor-service/
    catalog-service/
    media-service/
    admin-service/
  docs/
  scripts/
  docker-compose.yml
```

## Related Documents

- [README.md](/Users/woron/Documents/netiks-store/README.md): main project setup guide
- [INTERN_QUICKSTART_GUIDE.md](/Users/woron/Documents/netiks-store/docs/INTERN_QUICKSTART_GUIDE.md): simple run instructions for interns
- [PRD.md](/Users/woron/Documents/netiks-store/docs/PRD.md): product scope and requirements
- [TECHNICAL_PLAN.md](/Users/woron/Documents/netiks-store/docs/TECHNICAL_PLAN.md): technical implementation details
