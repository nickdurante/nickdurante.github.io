---
title: "How to Connect a Keycloak Identity Provider to Another Keycloak instance using OpenID Connect"
categories:
  - keycloak
  - federated
  - IDP
tags:
  - tech
comments: true
---

Tested on Keycloak Docker version 24.0.4.

## Prerequisites
Ensure Docker and Docker Compose are installed on your system.

## Step 1: Run Keycloak Instances
Run two Keycloak instances using Docker Compose. Adjust accordingly if only one is needed.

**docker-compose.yml:**
```yaml
services:
  keycloak_master:
    image: keycloak/keycloak:latest
    hostname: keycloak-master
    container_name: keycloak-master
    command: ["start-dev"]
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    ports:
      - "8080:8080"

  keycloak_slave:
    image: keycloak/keycloak:latest
    hostname: keycloak-slave
    container_name: keycloak-slave
    command: ["start-dev"]
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    ports:
      - "8081:8080"
```

## Step 2: Create Realms
- **Keycloak Master:** Create a realm named `master_realm`.
- **Keycloak Slave:** Create a realm named `slave_realm`.

## Step 3: Create Client for the Identity Provider in Slave
1. Navigate to `slave -> slave_realm -> clients`.
2. Create a client named `idp_client`.
3. Set "Valid Redirect URIs" to `*`.
4. Go to `Credentials` and regenerate the client secret. Copy the secret.

## Step 4: Setup Identity Provider in Master
1. Navigate to `master -> identity providers`.
2. Add a new provider of type `Keycloak OpenID Connect`.

**Configuration:**
- **Display Name:** (This will be shown on the button)
- **Authorization URL:** `http://localhost:8081/realms/slave_realm/protocol/openid-connect/auth`
- **Token URL:** `http://keycloak-slave:8080/realms/slave_realm/protocol/openid-connect/token`
  - **Note:** The Authorization URL is accessed by the browser (local network), while the Token URL is accessed internally (Docker network).
- **Client ID:** `idp_client`
- **Client Secret:** `<idp_client_secret>` (from the previous step)

## Step 5: Create Users
- **Keycloak Master:**
  1. Navigate to `master -> users`.
  2. Create a user named `master_user`.
  3. Set the password to `master_password`.
- **Keycloak Slave:**
  1. Navigate to `slave -> users`.
  2. Create a user named `slave_user`.
  3. Set the password to `slave_password`.

## Step 6: Create Client for Application
1. Navigate to `master -> clients`.
2. Create a client named `app_client`.
3. Go to `Credentials` and regenerate the client secret. Copy the secret.

## Note
Users that log in from the slave Keycloak will also be saved in the master Keycloak.

## TODO
Implement privilege mapping from slave to master.
