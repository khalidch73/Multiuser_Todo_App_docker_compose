## Step:01 add Dockerfile in main route dir
Dockerfile.dev
```bash
# Use an official Python runtime as a parent image
FROM python:3.11

LABEL maintainer="khalid_nawaz"
# Set the working directory in the container
WORKDIR /code
# Install system dependencies required for potential Python packages
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Poetry
RUN pip install poetry

# Copy the current directory contents into the container at /code
COPY . /code/

# Configuration to avoid creating virtual environments inside the Docker container
RUN poetry config virtualenvs.create false

# Install dependencies including development ones
RUN poetry install

# Make port 8000 available to the world outside this container
EXPOSE 8000

# Run the app. CMD can be overridden when starting the container
CMD ["poetry", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--reload"]
```
## Step:02 create compose.yaml in root dir
compose.yaml

```yaml

version: "3.9"

name: todo_api

services:
  # 01 api service
  api:
    build:
      context: ./todo_app
      dockerfile: Dockerfile.dev
    depends_on:
        - postgres_db
    ports:
      - "8000:8000"  # Expose container port 8000 to host port 8000  
    networks:
      - custome_network

  # 02 database service
  postgres_db:
    image: postgres:latest  # Use the official PostgreSQL image
    restart: always
    container_name: Postgres_Cont
    environment:
        - POSTGRES_USER=khalidNawaz
        - POSTGRES_PASSWORD=my_password
        - POSTGRES_DB=mydatabase
    ports:
        - '5433:5432'
    volumes:
        - postgres_db:/var/lib/postgresql/data
    networks:
      - custome_network

volumes:
  postgres_db:
    driver: local

networks:
  custome_network:  # Define the custom network
```

## Step:03 remove "sslmode": "require" from engine in db.py
engine = create_engine(
    connection_string, connect_args={"sslmode": "require"}, pool_recycle=300
)

```bash 
engine = create_engine(
    connection_string, connect_args={}, pool_recycle=300
)
```

## Step:04 validate your compose file
```bash 
docker compose config
```

## step:05 run compose file
```bash
docker compose up -d
```
### removed compose file
```bash
docker compose down
```
### stop compose file
```bash
docker compose stop
```
### start compose file 
```bash
docker compose start
```
### check running container
```bash
docker ps
```

