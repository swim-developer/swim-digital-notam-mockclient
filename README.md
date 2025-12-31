# SWIM DNOTAM Mock Client

> ⚠️ **Code Under Review** — This repository is part of a reference implementation currently under technical review. Source code will be available soon.

Interactive web client for testing and demonstrating SWIM DNOTAM Provider integration. Provides a complete UI for subscription management, AMQP message consumption, and real-time event visualization with Europe map overlay.

![Architecture](references/mockclient-architecture.svg)

## Overview

The Mock Client simulates an ANSP (Air Navigation Service Provider) consumer workstation, enabling operators to:

- **Authenticate** via Keycloak (OAuth2/OIDC)
- **Create/Manage Subscriptions** through Provider REST API
- **Connect to AMQP Broker** and consume DNOTAM events
- **Visualize Events** on an interactive Europe map
- **Inspect Messages** with syntax-highlighted AIXM XML viewer

## Features

| Feature | Description |
|---------|-------------|
| **Keycloak Integration** | OAuth2/OIDC authentication with token management |
| **Provider API Proxy** | Backend proxy with mTLS for secure Provider communication |
| **AMQP Consumer** | Vert.x-based client with per-user connection management |
| **Real-time Console** | Server-Sent Events (SSE) for live operation feedback |
| **Message Persistence** | MariaDB storage for received DNOTAM events |
| **Event Map** | SVG-based Europe map with event markers and clustering |
| **AIXM Viewer** | Prism.js syntax highlighting for XML inspection |
| **Event Injection** | REST endpoint for testing with custom AIXM payloads |

## Standards Alignment

| Standard | Status |
|----------|--------|
| SWIM-TI Yellow Profile | ✅ Implemented, ⏳ Pending Validation |
| AMQP 1.0 over TLS 1.2/1.3 | ✅ Implemented |
| mTLS Client Authentication | ✅ Implemented |
| OAuth2/OIDC (Keycloak) | ✅ Implemented |

## Technology Stack

- **Runtime**: Quarkus 3.30 (Java 21)
- **UI**: Qute Templates + Vanilla JavaScript
- **AMQP**: Vert.x AMQP Client
- **HTTP Client**: Vert.x Web Client
- **Database**: MariaDB (Hibernate ORM)
- **SSE**: Mutiny Reactive Streams

## UI Pages

| Page | Path | Description |
|------|------|-------------|
| Dashboard | `/ui` | Overview and authentication status |
| Provider API | `/ui/api` | Subscription management interface |
| Token | `/ui/token` | JWT token inspection and refresh |
| Console | `/ui/console` | Real-time operation logs (SSE) |
| AMQP | `/ui/amqp` | Broker connection and queue management |
| Messages | `/ui/messages` | Received DNOTAM events list |
| Subscriptions | `/ui/subscriptions` | Active subscription management |

## REST API

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/config/keycloak` | GET | Keycloak configuration for frontend |
| `/api/config/provider` | GET | Provider API URLs |
| `/api/status` | GET | mTLS and connection status |
| `/api/console/stream` | GET | SSE stream for console events |
| `/api/user/messages` | GET | User's received messages |
| `/api/messages/{id}/xml` | GET | Formatted XML view |
| `/api/messages/{id}/download` | GET | Download AIXM XML file |
| `/api/events/inject` | POST | Inject test DNOTAM event |
| `/api/map/events` | GET | SVG map with event markers |
| `/api/map/events.html` | GET | Interactive map page |

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `QUARKUS_HTTP_PORT` | `8080` | HTTP server port |
| `KEYCLOAK_URL` | `https://rhbk.apps.ocp4.masales.cloud` | Keycloak server URL |
| `KEYCLOAK_REALM` | `swim` | Keycloak realm name |
| `KEYCLOAK_CLIENT_ID` | `swim-public-client` | OAuth2 client ID |
| `SWIM_PROVIDER_API_URLS` | - | Comma-separated Provider API URLs |
| `SWIM_PROVIDER_AMQP_HOST` | - | AMQP broker hostname |
| `SWIM_PROVIDER_AMQP_PORT` | `443` | AMQP broker port |
| `PROXY_MTLS_KEYSTORE_PATH` | `certs/keystore.p12` | Client certificate keystore |
| `PROXY_MTLS_KEYSTORE_PASSWORD` | `changeit` | Keystore password |
| `PROXY_MTLS_KEYSTORE_TYPE` | `PKCS12` | Keystore type (PKCS12/JKS) |
| `PROXY_MTLS_TRUSTSTORE_PATH` | `certs/truststore.p12` | Trust store for CA certificates |
| `PROXY_MTLS_TRUSTSTORE_PASSWORD` | `changeit` | Truststore password |
| `PROXY_MTLS_TRUSTSTORE_TYPE` | `PKCS12` | Truststore type (PKCS12/JKS) |
| `MARIADB_HOST` | `localhost` | MariaDB hostname |
| `MARIADB_PORT` | `3306` | MariaDB port |
| `MARIADB_DATABASE` | `swim_client` | Database name |
| `MARIADB_USERNAME` | `swim` | Database username |
| `MARIADB_PASSWORD` | `swim` | Database password |

## Health Checks

| Endpoint | Description |
|----------|-------------|
| `/q/health/live` | Liveness probe |
| `/q/health/ready` | Readiness probe |
| `/q/health` | Combined health status |

## Operator Deployment

Deploy using the SWIM Operator:
```yaml
apiVersion: apps.swim-developer.github.io/v1alpha1
kind: SwimDnotamMockClient
metadata:
  name: dnotam-mockclient
spec:
  keycloak:
    url: "https://rhbk.apps.ocp4.masales.cloud/"
    realm: "swim"
    clientId: "swim-public-client"
  providerAPIURLs: "https://swim-dnotam-provider-mtls-swim-sandbox.apps.ocp4.masales.cloud"
  amqp:
    host: "provider-artemis-swim-sandbox.apps.ocp4.masales.cloud"
    port: 443
  mtls:
    enabled: true
    certsSecretName: "swim-dnotam-client-tls"
    passwordsSecretName: "swim-dnotam-mockclient-passwords"
```

## Local Development

### Start with dev services (MariaDB auto-provisioned)
./mvnw quarkus:dev

### Access UI
open http://localhost:8085/ui

### Build

#### JVM build
./mvnw clean package -DskipTests

#### Native build
./mvnw clean package -DskipTests -Pnative

#### Container image
./mvnw clean package -DskipTests -Dquarkus.container-image.build=true

## Event Injection

Test the system by injecting AIXM events directly:
```shell
curl -X POST http://localhost:8085/api/events/inject \
  -H "Content-Type: application/aixm+xml" \
  -d @sample-runway-closure.xml
```

## License
BSD 3-Clause License
