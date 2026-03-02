# NestJS Backend — Docker Practice

A hands-on practice project for containerizing a **NestJS** application using **Docker**. It includes two versions of Dockerfile to demonstrate the difference between a basic and an optimized build.

- **`Dockerfile`** — Basic single-stage build
- **`Dockerfile-v2`** — Optimized multi-stage build (smaller image size)

---

## 📁 Project Structure

```
NestJS-Backend-Docker/
├── src/
│   └── main.ts
├── Dockerfile          # Basic single-stage build
├── Dockerfile-v2       # Optimized multi-stage build
├── .dockerignore
└── ...
```

---

## 🐳 Basic Dockerfile (`Dockerfile`)

A simple **single-stage** build — easy to understand and great for learning.

```dockerfile
FROM node:20.20-alpine3.23

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["node", "dist/main"]
```

**Downside:** The final image includes both `devDependencies` and the original source files, resulting in a larger image than necessary.

### Build & Run

```bash
# Build image
docker build -t nestjs-app .

# Run container
docker run -p 3000:3000 nestjs-app
```

---

## 🚀 Optimized Dockerfile (`Dockerfile-v2`)

A **multi-stage** build — separates the build and production stages to significantly reduce the final image size.

```dockerfile
# Build stage
FROM node:20.20-alpine3.23 AS development

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install glob rimraf
RUN npm install --include=dev

COPY . .

RUN npm run build

# Production stage
FROM node:20.20-alpine3.23 AS production

ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --production

COPY . .

COPY --from=development /usr/src/app/dist ./dist

CMD ["node", "dist/main"]
```

**Benefits:**
- The production image **does not include** `devDependencies`
- Only the compiled `dist/` artifact is copied from the build stage
- Significantly smaller image size compared to the single-stage build

### Build & Run

```bash
# Build optimized image
docker build -f Dockerfile-v2 -t nestjs-app:optimized .

# Run container
docker run -p 3000:3000 nestjs-app:optimized
```

---

## 📊 Comparison

| Feature | `Dockerfile` | `Dockerfile-v2` |
|---|---|---|
| Build stages | 1 (single-stage) | 2 (multi-stage) |
| `devDependencies` in image | ✅ Yes | ❌ No |
| Image size | Larger | Smaller |
| Complexity | Simple | Moderate |
| Best for | Learning / Dev | Production |

---

## ⚙️ Running Locally (without Docker)

```bash
# Install dependencies
npm install

# Development mode
npm run start:dev

# Production mode
npm run build
npm run start:prod
```

---

## 🔧 Requirements

- [Node.js](https://nodejs.org/) >= 20
- [Docker](https://www.docker.com/) >= 24
