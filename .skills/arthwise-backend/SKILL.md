---
name: arthwise-backend
description: Core context and architecture for the Arthwise backend (ArthwiseServices). Trigger when the user asks about the backend, APIs, server, database, PM2, auth_microservice, or nse_service.
---

# Arthwise Backend Knowledge

## Architecture Overview
The backend is located in the `ArthwiseServices` directory. It uses an API Gateway pattern with two main microservices:

1. **API Gateway** (`ArthwiseServices/index.js`):
   - Node.js / Express server running on port 3000 (or `process.env.PORT`).
   - Handles Rate Limiting and Health Checks (`/healthz`).
   - Acts as a reverse proxy (`http-proxy-middleware`) routing requests to the underlying microservices based on URL paths.
   - Routes starting with `/api/user`, `/api/portfolio`, `/api/courses`, etc., are proxied to the Auth Microservice.
   - Routes starting with `/api/equity-list`, `/api/stock`, `/api/market-status` are proxied to the NSE Service.

2. **Auth Microservice** (`ArthwiseServices/auth_microservice/`):
   - Node.js (TypeScript/Express) service running on port 8000.
   - **Database**: MongoDB (via Mongoose).
   - **Caching & Pub/Sub**: Redis.
   - **Features**: User Authentication (Google OAuth, JWT), Portfolio management, Courses, Contests, Leaderboards, WebSocket connections (`socket.io`), Background Jobs (`BullMQ`).

3. **NSE Service** (`ArthwiseServices/nse_service/`):
   - Python FastAPI service running on port 8001.
   - **Features**: Fetches and caches stock market data (NIFTY, derivatives, commodities) using libraries like `nselib` and `yfinance`.
   - Heavily relies on Redis for caching API responses to reduce load on upstream NSE servers.

## Key Technologies
- **Node.js & Express**: Core framework for gateway and auth.
- **Python & FastAPI**: For data fetching and quantitative tasks.
- **MongoDB**: Primary database.
- **Redis**: Caching and Socket.io adapter.
- **PM2**: Process manager used for running and deploying the services.

## Common Workflows
- **Running Locally**:
  - Use the concurrent start script in `ArthwiseServices/package.json`: `npm run start` which boots the Gateway, Auth service, and NSE service simultaneously.
  - Ensure Redis and MongoDB are running locally or connected via `.env`.
- **Deployments**:
  - The application is typically run using PM2. Reference `ecosystem.config.js` or `ecosystem.config.cjs`.
  - Cloudflare Tunnels are used for exposing local instances to production environments.
