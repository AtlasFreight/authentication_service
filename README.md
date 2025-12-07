# Keycloak Cluster

This project sets up a **development** environment with:

- 2 instances of **Keycloak 24** (`kc1` and `kc2`)
- A **shared PostgreSQL database** (`keycloak-db`)
- Each Keycloak instance is accessible on a **different port** for testing.

> ⚠️ This setup is intended for **local development and testing**, not for production.

---

## Included Containers

| Service       | Image                             | Local Port | Description                        |
|---------------|----------------------------------|------------|------------------------------------|
| keycloak-db   | postgres:15                      | N/A        | Shared database for Keycloak       |
| kc1     | quay.io/keycloak/keycloak:24.0  | 8081       | Keycloak instance 1                |
| kc2     | quay.io/keycloak/keycloak:24.0  | 8082       | Keycloak instance 2                |

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
docker compose -p authentication_service -f deployment/docker-compose.yaml up -d
```

3. Verify that the containers are running:

```bash
docker-compose ps
```

---

## Accessing Keycloak
* Keycloak1: http://localhost:8081/admin/master/console/
* Keycloak2: http://localhost:8082/admin/master/console/
* Load Balancer: http://localhost:8000/admin/master/console/

Check `deployments/.env` for the default credentials.`

## Vault Configuration Variables
To set the Keycloak configuration variables in our microservices, we need to define
`spring.security.oauth2.resourceserver.jwt.issuer-uri` and
`spring.security.oauth2.resourceserver.jwt.jwk-set-uri`. So, if the microservice is running on our local machine, we can
set the following variables `KEYCLOAK_ISSUER_URI` and `KEYCLOAK_ISSUER_JWK` in the `application.properties` file:

```properties
# application.properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=${KEYCLOAK_ISSUER_URI:http://localhost/realms/atlas_freight}
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=${KEYCLOAK_ISSUER_JWK:http://localhost/realms/atlas_freight/protocol/openid-connect/certs}
```

As you can see we use `localhost` as the issuer URI and the JWK set URI. But if our microservice is dockerized, we can
set the variables using the following command:

```bash
vault kv put secret/application \
KEYCLOAK_ISSUER_JWK="http://keycloak_lb/realms/atlas_freight/protocol/openid-connect/certs" \
KEYCLOAK_ISSUER_URI="http://localhost/realms/atlas_freight"
```

---

## Useful Commands
* Stop the environment:

```bash
docker-compose down
```

* View logs for Keycloak1:

```bash
docker-compose logs -f kc1
```

* Restart Keycloak2:

```bash
docker-compose restart kc2
```

## Considerations

* **Cache and clustering**: Each Keycloak instance uses local cache (`KC_CACHE=local`), so changes are not automatically
synchronized between instances.
* **Consul or load balancer registration**: Not included in this setup.
