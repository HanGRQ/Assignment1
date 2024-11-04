## Docker Assignment - Agile Software Practice.

__Name:__ Sihan Ma

__Demo:__ https://youtu.be/16LUkyyby7Q

This repository contains the containerization of the mukti-container application illustrated below.

![](E:\assignment1-README\assignment1-README\images\arch.png)

### Database Seeding.

**Overview**: The database seeding process is designed to automatically populate MongoDB with initial movie data when the development environment is launched. This is facilitated through the `mongo-seed` container, which is built from a custom Dockerfile and uses a `seeding.json` file containing the seed data.

**Folder Structure**:

- `seeding/`
  - **Dockerfile**: Defines the build process for the `mongo-seed` container.
  - **seeding.json**: Contains the JSON-formatted movie data to be imported into MongoDB.

**Dockerfile**: The `Dockerfile` in the `seeding` folder contains instructions to build the `mongo-seed` image. Here's a breakdown:

```dockerfile
FROM mongo:8.0-rc
COPY seeding.json /seeding.json
CMD mongoimport --host mongo --db tmdb_movies --collection movies --type json --file /seeding.json --jsonArray --username admin --password password --authenticationDatabase admin
```

This configuration ensures that every time the development stack is launched, the MongoDB is pre-populated with movie data without manual intervention.

**docker-compose file**: The `mongo-seed` service is configured in the `docker-compose.yml` as follows:

```yaml
mongo-seed: 
  build: ./seeding
  container_name: mongo-seed
  depends_on:
    -mongo
  volumes:
    -/e/Agile/Assignment1/seeding/seeding.json:/seeding.json
  networks:
    -backend-db
```

### M.ulti-Stack.

To efficiently support both development and production environments, the project utilizes `docker-compose.yml` and `docker-compose.override.yml` files.

#### Development Stack

The **development stack** is designed to provide a comprehensive environment with additional tools and automation to enhance the development experience.

- **Core Services (from `docker-compose.yml`)**:
  - **Movies API**: Handles API requests for movie data.
  - **MongoDB**: Stores movie information.
  - **Redis**: Provides caching and rate-limiting.
- **Additional Development Tools (from `docker-compose.override.yml`)**:
  - **Mongo Express**: A web-based MongoDB administration interface, accessible at `http://localhost:8081`.
  - **Mongo Seed**: A service that automatically seeds MongoDB with initial data using the `seeding.json` file.

**Setup**: The development stack is launched using the command:

```bash
docker-compose up -d
```

This command combines configurations from both `docker-compose.yml` and `docker-compose.override.yml`, deploying all services, including `mongo-express` and `mongo-seed`.

#### Production Stack

The **production stack** is streamlined to ensure efficiency, security, and minimal resource usage by excluding non-essential services.

- **Core Services Only (from `docker-compose.yml`)**:
  - **Movies API**
  - **MongoDB**
  - **Redis**
- **Excluded Services**:
  - **Mongo Express**: Removed to avoid exposing database administration tools in production.
  - **Mongo Seed**: Disabled to prevent unnecessary seeding operations in a live environment.

**Setup**:

- Launch with:

  ```bash
  docker-compose -f docker-compose.yml up -d
  ```

  This command only uses `docker-compose.yml`, deploying the core services (`Movies API`, `MongoDB`, `Redis`) for production.
