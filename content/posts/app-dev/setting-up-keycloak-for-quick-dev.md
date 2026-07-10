---
title: "Setting Keycloak for quick development"
date: 2026-04-04
draft: false
comments: true
categories: ["spring-security"]
tags: ["microservices-security", "OAuth2", "oidc", "keycloak"]
---

**This is for quick tests. The data you config in this server will be deleted. when you delete the container. There are other posts which details how to persist data even after container is lost.**

**Keycloak Server**

#### 1. Set up Keycloak server locally #### 

First setup [Keycloak server locally through Docker](https://www.keycloak.org/getting-started/getting-started-docker)

```docker run -d -p 127.0.0.1:7080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.6.2 start-dev```

Since one of our services is runningunder 8080, I mapped to 7070. Internally keycloak is exposed at 8080.  

Once started, go to <http://localhost:7080>, log into the admin console using your credentials.

You will be taken to the only realm which you have currently which is *master realm*. A realm is a boundary where you set you client and user credentials. You can create any realms like, dev, qa, etc. Currently we are not going to make any more, and continue with *master realm*.

#### 2. Register client details inside KeyCloak for Client Credential grant flow ####

Click on *Clients* inside your admin dashboard. A few built in clients can be seen.

- Click 'Create client'. 
- Select **OpenID Connect** as the **CLient type** 
- Give a **Client ID** for your client credentials grant type flow, **Name**, and **Description**. 
- Click on **Next** button to move to the next screen. 
- Enable **Client Authentication** as we want our client to be authenticated with the Auth Server by providing it's own credentials, so as the Auth server can issue the *Access token*. 
f- Under **Authentication flow**, disable Standard flow* and *Direct Access grants*. 
- Select **Service account roles** as this enables support of *Client Credentials Grant'* for the client. 
- Click on **Next** button to move to the next screen. 
- Keep **Root URL** and **Home URL** empty, and click **Save**.

This takes us to the Client Details dashboard. Click on the **Credentials** tab and you can see Keycloakhas auto generated client secret for us.

Now we can check how the client can connect with the auth server to get an access token. The client needs to know the endpoint it has to invoke to it.

- Click on **Realm Settings** under **Configure** from the left navigation panel. At the end of the page you can click on **OpenID Endpoint Configuration** to open a configuration json which lists all the endpoits supported by your auth server. 
- Search for **token_endpoint**. This is the URL endpoint client has to invoke to get the access token. 
- Take this endpoint. We are going to test it inside a REST client like Postman. 
- Create a **POST** method with the endpoint as url. 
- Under **Body** select **x-www-form-urlencoded**. 
- Add **grant_type** for key and **client_credentials** for  value. *We are using `client crential` grant type flow. We can see this under the `grant_types_supported` key in the *OpenID Endpoint Configuration* we earlier opened. 
- Enter **client_id** and **client_secret** from the client we have created earlier under the same keys respectively. 
- Enter **scope** for value and and **openid email profile**. *NB: Remember the space.* These are scopes which are already provided by default by Keycloak. You can see them under **Clients -> Client scopes**, **openid** is transient as we created a client using OpenID Connect. If you need to create your own scopes, you can do that udner **Client scopes**. 
- You will receive *acces_token* and *id_toke* in response. *If you haven't added openid as scope you will only receive access token.*

Currenttly everything is setup from the client side and the Auth Server, Now we need to setup our API Gateway Server as a resource server.


