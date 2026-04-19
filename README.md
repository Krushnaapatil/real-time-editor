# 🖥️ Real-Time Collaborative Code Editor

A real-time collaborative code editor where multiple users can edit the same file simultaneously — like Google Docs, but for code. Built with React, Node.js, Yjs, Socket.io, Docker, and deployed on AWS ECS.

---

## 📸 What It Does

- Multiple users can join a shared editor session using a username
- All edits sync in real-time across every connected user
- A live sidebar shows who is currently in the session
- Built on **Yjs** (CRDT-based sync) + **Socket.io** (WebSockets) for conflict-free collaboration
- The entire app (frontend + backend) runs as a single Docker container

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js + Monaco Editor (VS Code's editor) |
| Real-Time Sync | Yjs (CRDT) + y-socket.io |
| Backend | Node.js + Express.js |
| WebSockets | Socket.io |
| Containerization | Docker (Multi-Stage Build) |
| Container Registry | AWS ECR (Elastic Container Registry) |
| Cloud Deployment | AWS ECS (Elastic Container Service) |

---

## 🧠 How It Works

### Real-Time Collaboration with Yjs
Yjs is a **CRDT (Conflict-free Replicated Data Type)** library. This means:
- Every user has a local copy of the document
- Edits are merged automatically without conflicts
- No central "locking" mechanism needed — everyone types freely

### Architecture Flow

```
User A (Browser)           User B (Browser)
     |                          |
  Monaco Editor             Monaco Editor
     |                          |
  y-socket.io               y-socket.io
     |                          |
     └──────── Socket.io ───────┘
                    |
             Node.js Server
             (YSocketIO handles sync)
                    |
            Serves React build
            from /public folder
```

The backend:
1. Serves the React frontend as static files from the `/public` folder
2. Handles WebSocket connections via Socket.io
3. Uses `YSocketIO` to sync Yjs documents between all connected clients

---

## 📁 Project Structure

```
Docker-AWS/
├── Frontend/
│   ├── src/
│   │   └── App.jsx          # Main React component
│   ├── package.json
│   └── vite.config.js
├── Backend/
│   ├── index.js             # Express + Socket.io server
│   └── package.json
└── dockerfile               # Multi-stage Docker build
```

---

## 💻 Local Development

### Prerequisites
- Node.js 20+
- Docker Desktop

### Run Without Docker

**Backend:**
```bash
cd Backend
npm install
node index.js
# Server running on http://localhost:3000
```

**Frontend:**
```bash
cd Frontend
npm install
npm run dev
# Dev server on http://localhost:5173
```

### Run With Docker

```bash
# Build the image
docker build -t server .

# Run the container
docker run -p 4000:3000 server

# App available at http://localhost:4000
```

---

## 🐳 Docker Setup (Multi-Stage Build)

The `dockerfile` uses a **multi-stage build** to keep the final image small and production-ready:

```dockerfile
# Stage 1 — Build the React frontend
FROM node:20-alpine AS frontend-builder
COPY ./Frontend /app
WORKDIR /app
RUN npm install
RUN npm run build          # Outputs to /app/dist

# Stage 2 — Run the backend + serve frontend
FROM node:20-alpine
COPY ./Backend /app
WORKDIR /app
RUN npm install
COPY --from=frontend-builder /app/dist /app/public   # Copy built frontend
```

**Why multi-stage?**
- Stage 1 installs all frontend dev dependencies and builds the React app
- Stage 2 only contains the backend + the compiled frontend files
- Result: a lean production image with no unnecessary build tools

The backend serves the React build as static files:
```js
app.use(express.static("public"))  // Serves /app/public = React build output
```

---

## ☁️ AWS Deployment

### Step 1 — Authenticate Docker with AWS ECR

```bash
aws ecr get-login-password --region ap-northeast-1 | docker login \
  --username AWS \
  --password-stdin 446931897427.dkr.ecr.ap-northeast-1.amazonaws.com
```

Output: `Login Succeeded`

### Step 2 — Build for Linux (AMD64)

Since AWS ECS runs Linux containers, build with the correct platform (important if you're on Windows/Mac ARM):

```bash
docker buildx build --platform linux/amd64 -t docker-aws/server .
```

### Step 3 — Tag the Image for ECR

```bash
docker tag docker-aws/server:latest \
  446931897427.dkr.ecr.ap-northeast-1.amazonaws.com/docker-aws/server:latest
```

### Step 4 — Push to ECR

```bash
docker push \
  446931897427.dkr.ecr.ap-northeast-1.amazonaws.com/docker-aws/server:latest
```

Output:
```
latest: digest: sha256:1679862924fc58dcbc7603577fe9f90f7126301149ff149c7f430e8602b17cb3 size: 1994
```

### Step 5 — Deploy on AWS ECS

1. Go to **AWS Console → ECS → Create Cluster**
2. Choose **Fargate** (serverless, no EC2 management)
3. Create a **Task Definition**:
   - Container image: `446931897427.dkr.ecr.ap-northeast-1.amazonaws.com/docker-aws/server:latest`
   - Port mapping: `3000`
4. Create a **Service** from the task definition
5. Set desired count (e.g., 1 for single instance, more for scale)
6. ECS pulls from ECR and runs your container automatically

---

## 🔑 Key Concepts Explained

### What is Docker?
Docker packages your app + all its dependencies into a **container** — a portable unit that runs the same everywhere, regardless of the host machine's OS or configuration.

### Containers vs Virtual Machines
| | Container | Virtual Machine |
|---|---|---|
| Startup | Seconds | Minutes |
| Size | MBs | GBs |
| OS | Shares host kernel | Full OS per VM |
| Isolation | Process-level | Hardware-level |

### What is ECR?
**Elastic Container Registry** — AWS's private Docker image registry. Like Docker Hub, but hosted on AWS and integrated with ECS.

### What is ECS?
**Elastic Container Service** — AWS's managed container orchestration platform. You give it a Docker image and it handles running, scaling, and restarting containers.

### What is Yjs + CRDT?
**Yjs** is a conflict-free data sync library. CRDTs guarantee that even if two users edit simultaneously, their changes merge correctly without data loss — no "last write wins" conflicts.

---

## 🏥 Health Check Endpoint

The backend exposes a health check route used by AWS ECS to verify the container is running:

```
GET /health
→ { "message": "ok", "success": true }
```

---

## 🚀 What You'll Learn From This Project

- What Docker is and why developers use it
- Difference between containers and virtual machines
- How to write a Dockerfile (`FROM`, `WORKDIR`, `COPY`, `RUN`)
- Multi-stage Docker builds for optimized images
- Real-time collaboration using Yjs (CRDTs) + Socket.io
- How to push Docker images to AWS ECR
- How to deploy containerized apps on AWS ECS
- How real-world systems scale for multiple users

---

## 📋 Project Timeline (Video Reference)

| Timestamp | Topic |
|---|---|
| 00:00 – 02:32 | Introduction |
| 02:32 – 07:40 | What We Are Going to Build |
| 07:40 – 18:46 | Setting Up the Monaco Editor |
| 18:46 – 24:57 | Creating Server with Yjs & Socket.io |
| 24:57 – 35:00 | Setting Up Yjs in React |
| 35:00 – 44:55 | Users in the Room (Part 1) |
| 44:55 – 55:49 | Users in the Room (Part 2) |
| 55:49 – 01:02:56 | Why We Use Docker |
| 01:02:56 – 01:07:04 | Installing Docker |
| 01:07:04 – 01:17:30 | Running Backend & Frontend on the Same Domain |
| 01:17:30 – 01:35:21 | Running Backend Server Using Docker |
| 01:35:21 – 01:45:00 | Multi-Stage Build |
| 01:45:00 – 01:48:32 | What are ECR & ECS? |
| 01:48:32 – 02:16:02 | Pushing Docker Image to ECR |
| 02:16:02 – 02:58:58 | Deploying on AWS ECS |
