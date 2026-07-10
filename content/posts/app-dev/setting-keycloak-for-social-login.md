---
title: "Enabling Facebook Login for a react website using Keycloak"
date: 2026-04-13
draft: false
comments: true
categories: ["spring-security"]
tags: ["microservices-security", "OAuth2", "oidc", "keycloak"]
---

I was working on a personal project using React for the frontend and Spring Boot for the backend. Since the application needed authentication, I decided to use social logins through Keycloak running locally inside Docker.

One thing that caught me off guard early on was how Keycloak behaves when started directly using the commands from the official documentation. While it works fine for quick experiments, the setup is ephemeral. The moment you remove the container or volume — something that happens quite often during development and debugging — everything inside Keycloak disappears, including realms, clients, users, and configuration changes. I learned that the hard way after having to recreate everything multiple times.

To avoid that pain, it is much better to run Keycloak with a proper `docker-compose.yaml` setup backed by a persistent PostgreSQL database.

Add the following configuration to your `docker-compose.yaml` file:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: <your_app_name>-postgres
    restart: always
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: keycloak
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  keycloak:
    image: quay.io/keycloak/keycloak:26.6.2
    container_name: <your_app_name>-keycloak
    command: start-dev
    restart: always
    environment:
      KC_DB: postgres
      KC_DB_URL_HOST: postgres
      KC_DB_URL_DATABASE: keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: keycloak

      KC_BOOTSTRAP_ADMIN_USERNAME: admin
      KC_BOOTSTRAP_ADMIN_PASSWORD: admin

      KC_HTTP_ENABLED: true
      KC_HOSTNAME_STRICT: false
      KC_PROXY: edge

    ports:
      - "7080:8080"

    depends_on:
      - postgres

volumes:
  postgres_data:
```
mapped Keycloak to port 7080 because my Spring microservices were already using port 8080.

Once the file is ready, start everything using ```docker compose up -d```

** Understanding the Important Keycloak Configurations 	**

```yaml
keycloak:
	environment:
		KC_HTTP_ENABLED: true
      	KC_HOSTNAME_STRICT: false
      	KC_PROXY: edge
```

### KC_HTTP_ENABLED: true ###

**What it does**

This setting allows Keycloak to accept traffic over plain HTTP.

During local development, most of us access Keycloak using something like: http://localhost:7080 instead of HTTPS. 
Without enabling HTTP support ,Keycloak expects secure HTTPS connections, which can lead to several frustrating issues such as redirect URI mismatches, secure cookie problems, and failed OAuth callbacks from providers like Facebook or Apple.

Enabling this setting keeps local development simple and avoids unnecessary HTTPS complications while building and testing the application.

In production, however, the setup is usually different. HTTPS is typically handled by a reverse proxy such as Nginx, Traefik, a cloud load balancer, or Kubernetes ingress. In those environments, HTTP should normally be disabled.

A typical production request flow looks like this: ```Browser → HTTPS Proxy → Keycloak```

### KC_HOSTNAME_STRICT: false ###

Keycloak normally prefers operating under a single canonical hostname, such as: ```auth.yourapp.com```

When strict hostname validation is enabled, requests coming from unexpected origins, ports, or domains may be rejected. During development, though, applications are often accessed from multiple places like:

- localhost:7080
- 127.0.0.1:7080
- frontend apps running on localhost:5173 (e.g., my web app)
- OAuth callback URLs from social login providers

Because of this, strict hostname validation can easily cause issues such as:

- invalid redirect_uri
- hostname mismatch errors
- blocked origins

Disabling strict hostname checks makes local development much smoother and removes a lot of unnecessary friction while testing authentication flows.

For production environments, it is better to explicitly define the hostname and enable strict validation for better security and predictable redirects.

Example:

```yaml
KC_HOSTNAME: auth.myapp.com
KC_HOSTNAME_STRICT: true
```

### KC_PROXY: edge ###

This configuration becomes very important when Keycloak is deployed behind a reverse proxy.

In many real-world deployments, HTTPS is terminated at a gateway or proxy layer before traffic reaches Keycloak internally over HTTP. Without proxy awareness, Keycloak may incorrectly assume requests are coming through plain HTTP and generate wrong redirect URLs.

For example, without proper proxy configuration, Keycloak might generate ```http://auth.yourapp.com``` instead of ```https://auth.yourapp.com```

This can break social logins, OAuth redirects, cookies, and JWT issuer validation.

The value edge tells Keycloak that HTTPS ends at the proxy layer and that internally it will still receive HTTP traffic.

A typical deployment flow looks like this:

```
Browser
   ↓ HTTPS
Nginx / Gateway
   ↓ HTTP
Keycloak
```
Even during local development, enabling this early helps maintain consistency with production architecture and prevents future issues when integrating providers like Facebook and Apple login.

**Summary**
| Setting                  | Purpose                            | Why Needed                                 |
| :---                     | :---                               | :---                                       |
| KC_HTTP_ENABLED=true     | Allow HTTP                         | Local development on localhost             |
| KC_HOSTNAME_STRICT=false | Disable strict hostname validation	| Easier local OAuth/callbacks               |
| KC_PROXY=edge            | Trust reverse proxy headers	    | Correct OAuth redirects and HTTPS handling |

**Recommended Local Dev Setup**

```
KC_HTTP_ENABLED: true
KC_HOSTNAME_STRICT: false
KC_PROXY: edge
```

**Recommended Production Setup Later**

```
KC_HTTP_ENABLED: false
KC_HOSTNAME: auth.yourapp.com
KC_HOSTNAME_STRICT: true
KC_PROXY: edge
```
with:

- HTTPS enabled at gateway/load balancer
- proper DNS
- SSL certificates
- Correct reverse proxy headers


------------------

Now we need To Configure In Keycloak

###  STEP 1 — Create Realm

Example:

*Realm Name*: your-app

A realm is basically: Separate security universe

### STEP 2 — Create React Client

Inside:

**Clients → Create client**

Configure:

| Setting               |	Value          |
| :---                  | :---           |
| Client Type	          | OpenID Connect |
| Client ID	            | your-app-web    |
| Name	                | YourApp Web    |
| Client Authentication | OFF            |
| Authorization	        | OFF            |
| Standard Flow	        | ON             |
| Direct Access Grants	| OFF            |

Important:

React SPA should be a Public Client NOT confidential.

### STEP 3 — Configure Redirect URLs

Inside client settings:

Valid Redirect URIs 

http://127.0.0.1.nip.io:5173/* (or your frontend port)

Valid Post Logout Redirect URIs

http://127.0.0.1.nip.io:5173/*

Web Origins

http://127.0.0.1.nip.io:5173 or: *

*during development only*. 
[what is http://127.0.0.1.nip.io:5173/* ?] (../what-is-nip-io.md/)

### STEP 4 — Configure Login Options

Inside:

**Realm Settings → Login**

Enable:

| Option	          | Value |
| :---              | :---  |
| User registration	| ON    |
| Forgot password	  | ON    |
| Remember me	      | ON    |
| Login with email	| ON    |

Optional:
Email verification
MFA later

### STEP 5 — Configure Facebook Login

Go to:

**Identity Providers → Facebook**

You need `Client ID` and `Client Secret`  from **Meta for Developers.** 

You need to register your app already in Facebook. Check how to [here](../create-facebook-app-in-meta-developer-console/).

You can get them under **Meta for Developers -> Your App -> App Settings -> Basic**

Then configure in Keycloak:

| Field	         | Value                  |
| :---           | :---                   |
| Client ID      | App ID	from Meta       |
| Client Secret  | App Secret	from Meta   |
| Default Scopes | `email public_profile` |

*NB: `email public_profile` should be exactlu=y like that*

In your **Meta App Dashboard**, under **App customization and requirements**, select your use case, for example **Customize the Authenticate and requestdata from users with Facebook Login use case** in this case. Select on **Settings**. 

In our local Keycloak Redirect URi shows something like http://localhost:7080/realms/vilka/broker/facebook/endpoint, but it is will not work correctly, so add http://127.0.0.1.nip.io:7080/realms/master/broker/facebook/endpoint under **Valid OAuth Redirect URIs**.

Again it won't allow it. I complains *HTTPS is required for all redirect URLs.*

That happens because Meta for Developers no longer allows plain `http://` redirect URIs for most Facebook Login apps. Meta expects OAuth callbacks to use `https://`.

// TODO - Continue from here. create tunnel.


### STEP 7 — Add Roles

Inside:

Realm Roles

Create:

USER
ADMIN
HOST

Later your Spring Gateway extracts these.

STEP 8 — Configure Spring Gateway

Your Gateway becomes:

OAuth2 Resource Server

NOT login manager.

In application.yml

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/vilka
```

That alone lets Spring validate Keycloak JWTs.

STEP 9 — React Integration

This is the important part.

DO NOT manually build OAuth redirects.

Use:

keycloak-js

Install:

npm install keycloak-js
Recommended React Structure
src/
 ├── auth/
 │    ├── keycloak.ts
 │    ├── AuthProvider.tsx
 │    └── useAuth.ts

keycloak.ts
```ts
import Keycloak from "keycloak-js";

const keycloak = new Keycloak({
  url: "http://localhost:8080",
  realm: "vilka",
  clientId: "vilka-web",
});

export default keycloak;
```

AuthProvider.tsx

```ts
import { ReactNode, useEffect, useState } from "react";
import keycloak from "./keycloak";

type Props = {
  children: ReactNode;
};

export default function AuthProvider({ children }: Props) {
  const [authenticated, setAuthenticated] = useState(false);

  useEffect(() => {
    keycloak
      .init({
        onLoad: "check-sso",
        pkceMethod: "S256",
      })
      .then((auth) => {
        setAuthenticated(auth);
      });
  }, []);

  if (!authenticated) {
    return (
      <button onClick={() => keycloak.login()}>
        Login
      </button>
    );
  }

  return <>{children}</>;
}
```

Wrap App

In main.tsx
```ts
<AuthProvider>
  <App />
</AuthProvider>
```

STEP 10 — Call Gateway APIs

Axios example:

```ts
import axios from "axios";
import keycloak from "./auth/keycloak";

const api = axios.create({
  baseURL: "http://localhost:8081",
});

api.interceptors.request.use((config) => {
  config.headers.Authorization =
    `Bearer ${keycloak.token}`;

  return config;
});

export default api;
```

What Happens During Login

When user clicks login:

React
  → redirects to Keycloak

Keycloak
  → shows login page

User clicks Apple

Keycloak
  → redirects to Apple

Apple
  → authenticates

Apple
  → returns to Keycloak

Keycloak
  → creates local session
  → issues JWT

React
  → receives token

Gateway
  → validates JWT

Important Production Notes
1. Use PKCE

Already enabled via:

pkceMethod: "S256"

Critical for SPAs.

2. Never Store Tokens In localStorage

Prefer:

memory
silent refresh

Keycloak adapter handles this.

3. Configure HTTPS Later

Apple login requires HTTPS in production.

Your Next Actual Coding Steps

Order matters:

Create realm
Create React client
Configure redirect URIs
Install keycloak-js
Add AuthProvider
Test username/password login
Configure Gateway JWT validation
Add Facebook provider
Add Apple provider
Add roles/authorization

Start with username/password first before social login.

That reduces debugging complexity massively.