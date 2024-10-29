## Agile Software Practice - Assignment 1.

## Overview

This repository hosts a Dockerized multi-container application for managing movie data via a RESTful API. The **Movies API** retrieves movie information from **MongoDB**, with **Redis** used for caching responses and applying rate limits to certain endpoints. Additionally, **Mongo Express** provides a web interface for MongoDB management, but is only available in the development environment. This project also includes automatic database seeding, enabled through a dedicated `mongo-seed` container, which pre-fills MongoDB with movie data during development.

## Table of Contents
- [Project Architecture](#project-architecture)
- [Environment Setup](#environment-setup)
  - [Development Stack](#development-stack)
  - [Production Stack](#production-stack)
- [Containers and Images](#containers-and-images)
  - [Movies API](#movies-api)
  - [MongoDB](#mongodb)
  - [Redis](#redis)
  - [Mongo Express (Development Only)](#mongo-express-development-only)
  - [Mongo Seed (Development Only)](#mongo-seed-development-only)
- [Database Seeding](#database-seeding)
- [Container Isolation](#container-isolation)
- [Setup Instructions](#setup-instructions)
  - [Starting the Development Environment](#starting-the-development-environment)
  - [Starting the Production Environment](#starting-the-production-environment)
- [API Testing](#api-testing)
- [Commit Log and Documentation](#commit-log-and-documentation)

## Project Architecture

The Movies API application architecture includes:
- **Movies API**: Retrieves movie data from **MongoDB**.
- **MongoDB**: Stores the movie data for the API.
- **Redis**: Caches responses from the Movies API and applies rate limits on the `/movies` endpoint.
- **Mongo Express**: A web interface for MongoDB administration, only available in the development environment.
- **Mongo Seed**: A dedicated service that pre-populates MongoDB with movie data during development.

### Docker Images Used
- `doconnor/movies-api:1.0`
- `mongo:8.0-rc`
- `redis:alpine`
- `mongo-express:1.0-20-alpine3.19`

## Environment Setup

### Development Stack
In the development environment:
- The entire stack is deployed, including **Movies API**, **MongoDB**, **Redis**, **Mongo Express**, and **Mongo Seed**.
- The database is automatically seeded with movie data using `mongo-seed`.

### Production Stack
In the production environment:
- Only **Movies API**, **MongoDB**, and **Redis** are deployed.
- Database seeding is disabled, and **Mongo Express** is not deployed to enhance security and efficiency.

## Containers and Images

### Movies API
- **Image**: `doconnor/movies-api:1.0`
- **Port**: `9000`
- **Environment Variables**:
  - `MONGODB_URI`: MongoDB URL (e.g., `mongodb://admin:password@mongo:27017`)
  - `REDIS_URI`: Redis URL (e.g., `redis://redis:6379`)
  - `ENABLE_WRITING_HANDLERS`: Set to `false`

### MongoDB
- **Image**: `mongo:8.0-rc`
- **Port**: `27017`
- **Environment Variables**:
  - `MONGO_INITDB_ROOT_USERNAME`: MongoDB root username
  - `MONGO_INITDB_ROOT_PASSWORD`: MongoDB root password

### Redis
- **Image**: `redis:alpine`
- **Port**: `6379`
- **Environment Variables**: None required

### Mongo Express (Development Only)
- **Image**: `mongo-express:1.0-20-alpine3.19`
- **Port**: `8081`
- **Environment Variables**:
  - `ME_CONFIG_MONGODB_URL`: MongoDB URL for Mongo Express connection

### Mongo Seed (Development Only)
- **Purpose**: Automatically seeds MongoDB with initial movie data during development.
- **Dockerfile**:
  ```dockerfile
  FROM mongo:8.0-rc
  COPY seeding.json /seeding.json
  CMD mongoimport --host mongo --db tmdb_movies --collection movies --type json --file /seeding.json --jsonArray --username admin --password password --authenticationDatabase admin
  ```
  - **Build Path**: `./seeding`
  - **Volumes**: `seeding.json` file is mounted to `/seeding.json` in the container.

## Database Seeding

The `mongo-seed` service uses a custom Dockerfile to import initial movie data from `seeding.json` into MongoDB, specifically for development purposes. This service automatically runs during development, ensuring the database is pre-populated without manual intervention.

## Container Isolation

To meet isolation requirements:
- **Movies API** and **Redis** are connected to the `backend-api` network, allowing API access to Redis but restricting access to MongoDB and Mongo Express.
- **Movies API** and **MongoDB** share the `backend-db` network, enabling API access to the database, while isolating **Redis** from **MongoDB**.
- **Mongo Express** is connected only to `backend-db` in the development environment for MongoDB administration.

## Setup Instructions

### Starting the Development Environment
To start the complete stack with Mongo Express and database seeding:

```bash
docker-compose up -d
```

This command will load both `docker-compose.yml` and `docker-compose.override.yml`, deploying all services including `mongo-seed` and **Mongo Express**.

### Starting the Production Environment
To start only the core services in production mode:

```bash
docker-compose -f docker-compose.yml up -d
```

This command only loads the `docker-compose.yml` file, deploying **Movies API**, **MongoDB**, and **Redis** without Mongo Express or seeding.

## API Testing

To test the Movies API, MongoDB data, Redis caching, and Mongo Express, follow these steps:

1. **Check MongoDB Data via Mongo Express** (`Mongo Express` on port 8081):

   In the development environment, Mongo Express is accessible for MongoDB administration. Open a browser and navigate to:

   ```
   http://localhost:8081
   ```

   - This will display Mongo Express, where you can browse MongoDB collections and verify that the `movies` collection is populated with data.

2. **Check API Connection** (Root endpoint on port 9000):

   Open a browser to access the root endpoint to verify Movies API connectivity:

   ```bash
   http://localhost:9000
   ```

   - Expected result: A "connected ping" message indicating that the Movies API is running.

3. **Get All Movies** (`/movies` endpoint on port 9000):

   ```bash
   http://localhost:9000/movies
   ```
   - This command retrieves all movies stored in MongoDB through the Movies API.
   - **Redis Caching**: The response will be cached by Redis; repeated requests should respond faster as data is retrieved from the cache.
   - **Rate-Limiting**: If the rate limit is active, repeated requests may trigger a `429 Too Many Requests` status, preventing excessive requests.

4. **Retrieve Movie by ID** (`/movies/:id` endpoint on port 9000):

   To view details for a specific movie, replace `<movie_id>` with an actual movie ID from your data:

   ```bash
   http://localhost:9000/movies/<movie_id>
   ```
   - This command retrieves detailed information about a specific movie based on its ID, providing JSON output.
   - **Note**: Ensure that the movie ID exists in the MongoDB collection, as an invalid ID will result in a "not found" response.

## Commit Log and Documentation

The Git log documents each stage of development with clear commit messages, summarizing tasks completed. The README provides a complete and accurate overview of the setup, architecture, and API, ensuring full documentation for the work carried out on this assignment.
