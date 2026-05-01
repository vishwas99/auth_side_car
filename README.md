# auth_side_car
Public Mirror for Auth Side Car with Built images for Linux, Mac - x64 and ARM

Markdown# Sidecar Auth: Consumer Integration Guide

This repository contains the pre-built **Sidecar Auth** binary and configuration templates. This sidecar acts as a security layer that sits in front of your microservices to handle **JWT validation**, **multi-algorithm encryption**, and **automatic token refreshing**.

## Quick Start

1. **Download the Bundle**: Ensure the `sidecar-auth` binary and `auth-config.yaml` are in the same folder.
2. **Configure your Backend**: Update `targetUrl` in `auth-config.yaml` to point to your application.
3. **Set your Keys**: Place your `.pem` files or HMAC secret in the config.
4. **Run**:
   
   ```bash
   ./sidecar-auth
   
## Configuration (auth-config.yaml)

The sidecar looks for a file named auth-config.yaml in its execution directory.YAMLport: "8080"                 # Port the sidecar listens on
targetUrl: "http://app:9090" # Your actual application address
authMode: "STATELESS"        # Options: STATELESS, HYBRID
encryptionType: "RSA"        # Options: RSA, HMAC, EDDSA

# Algorithm Specifics
publicKeyPath: "secrets/public.pem"
privateKeyPath: "secrets/private.pem"
jwt:
  issuer: "sidecar-auth"
  expiryMinutes: 15
  secret: "your-hmac-secret-here"

# Path Mapping
endpoints:
  # auth_side_car
  Public mirror for Sidecar Auth with pre-built binaries for Linux and macOS (x64 & ARM).

  # Sidecar Auth — Consumer Integration Guide

  This repository contains the pre-built Sidecar Auth binary and configuration templates. The sidecar acts as a security layer that sits in front of your microservices to handle JWT validation, multi-algorithm encryption, and automatic token refreshing.

  ## Quick start

  1. Download the bundle: ensure the `sidecar-auth` binary and `auth-config.yaml` are in the same folder (for example, the project `bin/` directory).
  2. Configure your backend: update `targetUrl` in `auth-config.yaml` to point to your application.
  3. Set your keys: place your `.pem` files or HMAC secret referenced by the config into the `secrets/` folder.
  4. Run the sidecar:

  ```bash
  ./sidecar-auth
  ```

  ## Configuration (auth-config.yaml)

  The sidecar looks for a file named `auth-config.yaml` in its execution directory. Example values:

  ```yaml
  port: "8080"                 # Port the sidecar listens on
  targetUrl: "http://app:9090" # Your actual application address
  authMode: "STATELESS"        # Options: STATELESS, HYBRID
  encryptionType: "RSA"        # Options: RSA, HMAC, EDDSA

  # Algorithm specifics
  publicKeyPath: "secrets/public.pem"
  privateKeyPath: "secrets/private.pem"
  jwt:
    issuer: "sidecar-auth"
    expiryMinutes: 15
    secret: "your-hmac-secret-here" # used for HMAC mode

  # Path mapping
  endpoints:
    login: "/api/login"
    signup: "/api/signup"
    refresh: "/api/verify-refresh"
  ```

  ## Security architecture

  The sidecar acts as a gatekeeper. Configure your application to accept traffic only from the sidecar's IP or local interface to prevent bypassing authentication.

  How it handles requests:

  - Unprotected routes: `/api/login` and `/api/signup` are proxied directly. The sidecar intercepts successful login responses and issues a signed JWT.
  - Protected routes: every other request must include an `Authorization: Bearer <token>` header.
  - Header injection: once verified, the sidecar forwards the request to your app with an injected `X-User-ID` header. Your app should use this header to identify the user.

  ### Auto-refresh flow

  To avoid forcing users to re-login, the sidecar supports a silent-refresh flow.

  Client requirements:

  - If an access token is expired, the client should still send the expired token in the `Authorization` header and include a refresh UUID in `X-Refresh-Token`.

  Sidecar response:

  - If the refresh is successful, the sidecar returns the requested data and includes a new token in the response header `X-New-Access-Token`.

  Clients should check that header and update their stored token accordingly.

  ### Header reference

  | Header | Required | Origin | Description |
  |--------|----------|--------|-------------|
  | Authorization | Yes | Client | Must be `Bearer <JWT>` |
  | X-Refresh-Token | Optional | Client | Required for auto-refresh on expiry |
  | X-User-ID | No | Sidecar | Injected for your backend to read |
  | X-New-Access-Token | No | Sidecar | Returned to client upon successful rotation |

  ## Deployment with Docker

  You can bundle this sidecar into your container stack using a simple Dockerfile:

  ```dockerfile
  FROM alpine:latest
  WORKDIR /app
  COPY sidecar-auth .
  COPY auth-config.yaml .
  # Ensure your secrets folder is copied if using RSA/EdDSA
  COPY secrets/ ./secrets/
  CMD ["./sidecar-auth"]
  ```

  ## Files in this repo

  - `bin/` — contains `sidecar-auth` binary and `auth-config.yaml` (if distributed)
  - `secrets/` — recommended location for private/public keys and other secrets

  ## Next steps

  - Verify the `sidecar-auth` binary in `bin/` is executable: `ls -l bin/sidecar-auth`
  - Edit `bin/auth-config.yaml` to match your environment and start the sidecar.

  If you'd like, I can also: add example `auth-config.yaml` to the repo, add a basic Docker Compose example, or validate that `bin/sidecar-auth` exists and is executable on your machine.
