# Keycloak Docker Compose (Development)

This project sets up a **development** environment with:

- 2 instances of **Keycloak 24** (`keycloak1` and `keycloak2`)
- A **shared PostgreSQL database** (`keycloak-db`)
- Each Keycloak instance is accessible on a **different port** for testing.

> ⚠️ This setup is intended for **local development and testing**, not for production.

---

## Included Containers

| Service       | Image                             | Local Port | Description                        |
|---------------|----------------------------------|------------|------------------------------------|
| keycloak-db   | postgres:15                      | N/A        | Shared database for Keycloak       |
| keycloak1     | quay.io/keycloak/keycloak:24.0  | 8081       | Keycloak instance 1                |
| keycloak2     | quay.io/keycloak/keycloak:24.0  | 8082       | Keycloak instance 2                |

---

## Prerequisites

- Docker >= 24
- Docker Compose >= 1.29
- External Docker network `atlasfreight-net` created:

```bash
docker network create atlasfreight-net
```

---

## Starting the Environment

1. Clone this project or copy the `docker-compose.yml`.
2. Start the containers:

```bash
docker-compose up -d
```

3. Verify that the containers are running:

```bash
docker-compose ps
```

---

## Accessing Keycloak
* Keycloak1: http://localhost:8081/admin/master/console/
* Keycloak2: http://localhost:8082/admin/master/console/

Admin username/password:
```text
admin / admin
```

---

## Technical Details
* **PostgreSQL** is used as the central database (`keycloak-db`).
* Each Keycloak instance runs `start-dev` with the following options:

```text
--hostname-strict=false
--db=postgres
--db-url=jdbc:postgresql://keycloak-db:5432/keycloak
--db-username=keycloak
--db-password=secret
```

> Note: Although `start-dev` normally uses H2, it can connect to PostgreSQL if the URL, username, and password are
provided correctly.

* Each container connects to the Docker network `atlasfreight-net`.
* **PostgreSQL** data is persisted in a volume named `keycloak-db_data`.

---
 
## Useful Commands
* Stop the environment:

```bash
docker-compose down
```

* View logs for Keycloak1:

```bash
docker-compose logs -f keycloak1
```

* Restart Keycloak2:

```bash
docker-compose restart keycloak2
```

* Access the database:

```bash
docker exec -it keycloak-db psql -U keycloak -d keycloak
```

## Considerations

* **Cache and clustering**: Each Keycloak instance uses local cache (`KC_CACHE=local`), so changes are not automatically
synchronized between instances.
* **Consul or load balancer registration**: Not included in this setup.
* **Development mode**: `start-dev` is recommended only for testing and quick development. For production, use `start --optimized`.