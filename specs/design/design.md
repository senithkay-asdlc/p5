---
skillsApplied:
  - high-level-architecture
  - go
  - openapi-conventions
---

# Design Document

## Overview

This system is a minimal "Hello World" API: a single, publicly accessible
backend service that returns a greeting message and reports its own health.
It exists purely as a clean, end-to-end reference for building, deploying,
and verifying a service on the platform.

## Components

- **hello-api** (`service`) — serves the public greeting endpoint and a
  health-check endpoint; owns no other responsibility.

## Capabilities

### hello-api

- **Greeting** — respond to `GET /hello` with HTTP 200 and a JSON body
  `{"message": "Hello, World!"}`, `Content-Type: application/json`.
- **Health check** — respond to `GET /health` with HTTP 200 while the
  service is running normally, so the platform's liveness/readiness probe
  can verify it.
- **Error handling** — return `404 Not Found` for unknown routes, `405
  Method Not Allowed` for unsupported methods on known routes, and a generic
  `500 Internal Server Error` (with no internal details leaked) for
  unexpected failures.

## Data model

No persistent entities. The service is stateless and holds no data beyond
the static greeting message.

## Roles & access

- **Anonymous client** — any caller, unauthenticated. Can call `GET /hello`
  and `GET /health` freely; the API requires no login and no roles.

## Interactions

No component-to-component or external integrations. `hello-api` is a single,
self-contained service called directly by clients through the platform
gateway; it makes no outbound calls.

## Data flow

1. A client sends `GET /hello` to `hello-api`.
2. `hello-api` responds immediately with `200 OK`,
   `Content-Type: application/json`, and body `{"message": "Hello, World!"}`.
3. Separately, an operator or the platform's probe sends `GET /health`;
   `hello-api` responds `200 OK` while healthy.
4. Any unknown route or unsupported method instead receives a `404`/`405`;
   unexpected server faults return a generic `500` without internal detail.
