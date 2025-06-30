# PixelCast Docker Setup

This guide will help you run the entire PixelCast application (backend, frontend, and SRS streaming server) using Docker Compose.

## Prerequisites

- Docker and Docker Compose installed on your system
- Port 3000, 3001, 1935, 1985, 8000, 8001, and 8080 available on your host machine

## Quick Start

1. **Create Environment File**

   Create a `.env` file in the root directory with the following content:

   ```env
   # Backend Configuration
   BACKEND_PORT=3001
   NODE_ENV=production

   # SRS Configuration
   SRS_SERVER_IP=srs
   SRS_HTTP_API_URL=http://srs:1985
   SRS_RTMP_URL=rtmp://srs:1935
   SRS_HLS_URL=http://srs:8080
   SRS_WEBRTC_URL=http://srs:8000

   # Frontend Configuration (Public variables for client-side)
   NEXT_PUBLIC_BACKEND_URL=http://localhost:3001
   NEXT_PUBLIC_SRS_HTTP_API_URL=http://localhost:1985
   NEXT_PUBLIC_SRS_RTMP_URL=rtmp://localhost:1935
   NEXT_PUBLIC_SRS_HLS_URL=http://localhost:8080
   NEXT_PUBLIC_SRS_WEBRTC_URL=http://localhost:8000

   # WebRTC Configuration
   CANDIDATE=localhost

   # Logging
   LOG_LEVEL=info

   # CORS Configuration
   CORS_ORIGIN=http://localhost:3000
   ```

2. **Build and Run**

   ```bash
   # Build and start all services
   docker-compose up --build

   # Or run in detached mode
   docker-compose up --build -d
   ```

3. **Access the Application**

   - Frontend: <http://localhost:3000>
   - Backend API: <http://localhost:3001>
   - SRS HTTP API: <http://localhost:1985>
   - SRS HTTP Server: <http://localhost:8080>

## Services Overview

### SRS (Simple Realtime Server)

- **Container**: `pixelcast-srs`
- **Ports**: 1935 (RTMP), 1985 (HTTP API), 8080 (HTTP Server), 8000 (WebRTC UDP), 8001 (WebRTC TCP)
- **Purpose**: Handles streaming protocols (RTMP, HLS, WebRTC)

### Backend

- **Container**: `pixelcast-backend`
- **Port**: 3001
- **Purpose**: Deno-based API server
- **Dependencies**: SRS

### Frontend

- **Container**: `pixelcast-frontend`
- **Port**: 3000
- **Purpose**: Next.js web application
- **Dependencies**: Backend, SRS

## Environment Variables

### Required Variables

- `BACKEND_PORT`: Port for the backend service (default: 3001)
- `NEXT_PUBLIC_BACKEND_URL`: Backend URL accessible from the browser
- `NEXT_PUBLIC_SRS_*`: SRS URLs accessible from the browser

### Optional Variables

- `NODE_ENV`: Environment mode (development/production)
- `LOG_LEVEL`: Logging level (info, debug, error)
- `CORS_ORIGIN`: Allowed CORS origins

## Docker Commands

```bash
# Start services
docker-compose up

# Start services in background
docker-compose up -d

# Stop services
docker-compose down

# Rebuild and start
docker-compose up --build

# View logs
docker-compose logs

# View logs for specific service
docker-compose logs frontend
docker-compose logs backend
docker-compose logs srs

# Shell into a container
docker-compose exec backend sh
docker-compose exec frontend sh
```

## Troubleshooting

### Port Conflicts

If you encounter port conflicts, you can modify the port mappings in `docker-compose.yml`:

```yaml
ports:
  - "3001:3001"  # Change left side to available port
```

### Container Communication

- Services communicate using container names (e.g., `backend`, `srs`, `frontend`)
- External access uses `localhost` or your host IP

### Logs

Check container logs for troubleshooting:

```bash
docker-compose logs -f [service-name]
```

## Development Mode

For development, you might want to use volume mounts for hot reloading:

```yaml
volumes:
  - ./pixel-cast-frontend:/app
  - /app/node_modules
  - /app/.next
```

## Production Considerations

1. **Environment Variables**: Use secure values for production
2. **SSL/TLS**: Configure reverse proxy (nginx/traefik) for HTTPS
3. **Resource Limits**: Add resource constraints to containers
4. **Health Checks**: Implement health checks for better reliability
5. **Secrets Management**: Use Docker secrets or external secret management

## Network Architecture

All services run on the `pixelcast_network` bridge network, allowing internal communication while exposing necessary ports to the host.
