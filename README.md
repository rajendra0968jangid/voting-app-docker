## 🗳️ Voting App Docker (Without Docker Compose)

Run a multi-container Docker application manually — without using Docker Compose.

## 📌 Project Overview

This project demonstrates how to run a multi-container Docker application manually by creating everything step-by-step instead of using Docker Compose.

Instead of running:

docker compose up -d

## We will:

✅ Build images manually

✅ Create networks manually

✅ Create volumes manually

✅ Run containers one by one

✅ Manage dependencies manually

This project is ideal for learning Docker fundamentals and interview preparation.



## 🏗️ Project Architecture
Service	Description	Port \
vote	Frontend Voting App	8080 \
result	Result Dashboard	8081 \
worker	.NET Background Worker	— \
redis	In-memory message queue	— \
db	PostgreSQL Database	— \
seed	Optional database seeder	— \


## 🌐 Networks Used

front-tier
\
back-tier



💾 Volume Used

db-data → PostgreSQL persistent storage \



🚀 Manual Setup Instructions


## Step 1 — Create Networks

docker network create front-tier 
\
docker network create back-tier



**Verify:**


docker network ls


## Step 2 — Create Volume



docker volume create db-data


**Verify:**


docker volume ls


## Step 3 — Run Redis



docker run -d \
  --name redis \
  --network back-tier \
  -v $(pwd)/healthchecks:/healthchecks \
  --health-cmd="/healthchecks/redis.sh" \
  --health-interval=5s \
  redis:alpine



## Step 4 — Run PostgreSQL



docker run -d \
  --name db \
  --network back-tier \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -v db-data:/var/lib/postgresql/data \
  -v $(pwd)/healthchecks:/healthchecks \
  --health-cmd="/healthchecks/postgres.sh" \
  --health-interval=5s \
  postgres:15-alpine

**Check logs:**

docker logs -f db


## Step 5 — Build Vote Image



cd vote
\
docker build -t vote-app --target dev .
\
cd ..


## Step 6 — Run Vote Container



docker run -d \
  --name vote \
  --network front-tier \
  -p 8080:80 \
  -v $(pwd)/vote:/usr/local/app \
  vote-app

**Connect it to back-tier:**

docker network connect back-tier vote


## Step 7 — Build Result Image



cd result
\
docker build -t result-app .
\
cd ..


## Step 8 — Run Result Container



docker run -d \
  --name result \
  --network front-tier \
  -p 8081:80 \
  -p 127.0.0.1:9229:9229 \
  -v $(pwd)/result:/usr/local/app \
  result-app \
  nodemon --inspect=0.0.0.0 server.js

**Connect it to back-tier:**

docker network connect back-tier result


## Step 9 — Build Worker Image



cd worker
\
docker build -t worker-app .
\
cd ..


## Step 10 — Run Worker Container



docker run -d \
  --name worker \
  --network back-tier \
  worker-app


## Step 11 — (Optional) Run Seeder



**Build Seeder:**

cd seed-data
\
docker build -t seed-app .
\
cd ..

**Run Seeder:**

docker run --rm \
  --name seed \
  --network front-tier \
  seed-app



## 🌍 Access Applications


**Application	URL**


Vote App	http://localhost:8080
\
Result App	http://localhost:8081
\
\
📋 Container Startup Order

## Since Docker Compose is NOT used, start containers in this order:

redis

db

vote

result

worker

## ⚠️ Health checks are NOT automatically managed.
**If services fail due to timing issues, restart them manually.**

**🧹 Cleanup Commands**
\
Stop Containers

docker stop vote result worker redis db

Remove Containers

docker rm vote result worker redis db

Remove Networks

docker network rm front-tier back-tier

Remove Volume

docker volume rm db-data

## 🎯 Why Use Docker Compose Instead?

Docker Compose provides:

Automatic dependency handling

Automatic network creation

Easier multi-container management

Single command startup

Better for production use

Example:

docker compose up -d

## 🛠️ Requirements

Docker 20+

Docker BuildKit (optional)

Linux / macOS / Windows

## 👨‍💻 Author

**Rajendra Jangid**\
Full Stack Developer (MERN + DevOps)