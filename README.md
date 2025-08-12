# Guestbook Application
## Overview
The Guestbook is a simple web application written in Go (Golang) that allows users to submit and view entries via a web interface. It serves a frontend (HTML, CSS, JavaScript) and optionally connects to a Redis database to store entries persistently. The application is containerized with Docker and can be deployed on Fly.io or in a Kubernetes environment (e.g., IBM Cloud Kubernetes Service). This project includes two versions: v1 and v2, with v2 featuring an updated title (e.g., "Lavanya's Guestbook - v2") and adjusted resource limits.
## Features

Submit and display guestbook entries through a web UI.
Built with Go for the backend, serving static assets (index.html, script.js, style.css, jquery.min.js).
Supports Redis for persistent storage (optional, can use in-memory storage).
Containerized with Docker for easy deployment.
Deployable on Fly.io (free tier) or Kubernetes.
Supports Kubernetes features like rolling updates and autoscaling.

## Repository Structure
guestApp/
├── Dockerfile        # Docker configuration for building the Go app
├── main.go           # Main Go application code
├── go.mod            # Go module dependencies
├── go.sum            # Go module checksums
├── index.html        # Frontend HTML (updated for v2)
├── script.js         # Frontend JavaScript
├── style.css         # Frontend CSS
├── jquery.min.js     # jQuery library
├── deployment.yml    # Kubernetes deployment manifest (for lab environment)
├── fly.toml          # Fly.io configuration
├── .gitignore        # Git ignore file
├── LICENSE           # License file
└── README.md         # This file

## Prerequisites

- ** Go: Version 1.18 or later (for local development).
Docker: For building and running the containerized app.
Git: For version control and pushing to GitHub.
Fly.io Account: For free cloud hosting (sign up at fly.io).
GitHub Account: For hosting the guestApp repository.
Redis (optional): For persistent storage (e.g., Upstash free tier or Fly.io Redis app).
Kubernetes CLI (kubectl): For lab environment deployment (Skills Network or IBM Cloud).
IBM Cloud CLI: For lab environment (if using IBM Cloud Container Registry).

## Setup Instructions
Local Development

Clone the Repository:
git clone https://github.com/yourusername/guestApp.git
cd guestApp


Install Dependencies:
go mod tidy


Run Locally:

Without Redis (in-memory storage, if modified):go run main.go


With Redis (set environment variables):export REDIS_HOST=localhost
export REDIS_PORT=6379
docker run -d -p 6379:6379 redis:7
go run main.go


Access at http://localhost:3000.


Build Docker Image:
docker build -t guestbook:v2 .
docker run -p 3000:3000 guestbook:v2



Fly.io Deployment

Set Up Fly.io CLI:

Install flyctl:
macOS: brew install flyctl
Linux: curl -L https://fly.io/install.sh | sh
Windows: Download from fly.io/docs/flyctl/install


Log in:flyctl auth login




Create Fly.io App:
flyctl apps create --name guestapp-yourusername
flyctl regions set fra


Deploy Redis (if needed):

Create a Redis app:flyctl apps create --name guestapp-redis-yourusername
flyctl deploy --image redis:7 --app guestapp-redis-yourusername --no-cache


Update fly.toml with Redis host:[env]
  PORT = "3000"
  REDIS_HOST = "guestapp-redis-yourusername.internal"
  REDIS_PORT = "6379"




Deploy Guestbook App:

Configure fly.toml in the repo root:app = "guestapp-yourusername"

[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "3000"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0
  processes = ["app"]

[[vm]]
  cpu_kind = "shared"
  cpus = 1
  memory_mb = 256


Commit and push:git add fly.toml
git commit -m "Add fly.toml for Fly.io"
git push origin main


Deploy:flyctl launch --no-deploy
flyctl deploy




Access the App:

Open the app:flyctl open


Or visit https://guestapp-yourusername.fly.dev.
Test submitting entries in the Guestbook.


Monitor Logs:
flyctl logs



Kubernetes Deployment (Lab Environment)
For the Skills Network lab or IBM Cloud Kubernetes Service:

Set Namespace:
export MY_NAMESPACE=sn-labs-kutiyoung


Build and Push Docker Image:
docker build -t us.icr.io/$MY_NAMESPACE/guestbook:v2 .
docker push us.icr.io/$MY_NAMESPACE/guestbook:v2


Apply Deployment:

Update deployment.yml:apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  replicas: 1
  selector:
    matchLabels:
      app: guestbook
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - image: us.icr.io/sn-labs-kutiyoung/guestbook:v2
        imagePullPolicy: Always
        name: guestbook
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: REDIS_HOST
          value: "redis"
        - name: REDIS_PORT
          value: "6379"
        resources:
          limits:
            cpu: 5m
            ephemeral-storage: 5Gi
            memory: 512Mi
          requests:
            cpu: 2m
            ephemeral-storage: 512Mi
            memory: 128Mi


Apply:kubectl apply -f deployment.yml




Deploy Redis:
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:latest
        ports:
        - containerPort: 6379
EOF


Access the App:
kubectl port-forward deployment.apps/guestbook 3000:3000


Use the Skills Network Toolbox to launch the app on port 3000.



## Troubleshooting

Fly.io Deployment Fails:
Ensure Dockerfile, main.go, and go.mod are in the repo root.
Check logs: flyctl logs.
Redeploy with: flyctl deploy --no-cache.


Redis Connection Issues:
Verify Redis app status: flyctl status --app guestapp-redis-yourusername.
Use Upstash for a free external Redis instance if needed.


Kubernetes Errors:
Check pod status: kubectl get pods -l app=guestbook.
Describe pods: kubectl describe pod -l app=guestbook.



Peer-Graded Assignment Tasks
For the Skills Network lab:

Dockerfile.png: Screenshot the Dockerfile.
crimages.png: Run ibmcloud cr images to show images.
app.png: Screenshot the v1 app after fixing Redis.
hpa.png, hpa2.png: Set up autoscaling:kubectl autoscale deployment guestbook --cpu-percent=50 --min=1 --max=10
kubectl get hpa


upguestbook.png: Screenshot docker build/push for v2.
deployment.png: Screenshot kubectl apply -f deployment.yml for v2.
up-app.png: Screenshot the v2 app.
rev.png, rs.png: Perform rollback:kubectl rollout history deployment guestbook
kubectl rollout undo deployment guestbook



License
MIT License (see LICENSE file).
