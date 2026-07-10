Meta for Developers no longer accepts plain http:// redirect URIs for most Facebook Login apps. Meta now expects OAuth callbacks to use https://.

So while this Keycloak broker endpoint works perfectly fine locally from Keycloak’s perspective:

http://localhost:7080/realms/vilka/broker/facebook/endpoint

Meta blocks it because it isn’t using HTTPS.

For local development, you have a few options.

Option 1 — Use an HTTPS tunnel (recommended)

The easiest approach is to expose your local Keycloak instance through a secure HTTPS tunnel.

Popular tools for this are:

ngrok
Cloudflare Tunnel

I initially tried ngrok, but my antivirus kept flagging it as suspicious, so I switched to Cloudflare Tunnel instead.

Cloudflare Tunnel (cloudflared)

Cloudflare Tunnel is a solid alternative that usually avoids antivirus issues and gives you a public HTTPS URL pointing to your local Keycloak instance.

Step 1 — Install cloudflared

Using winget:

winget install Cloudflare.cloudflared

Verify the installation:

cloudflared --version
Step 2 — Start your Keycloak container

Example:

docker run -d ^
-p 7080:8080 ^
-e KC_BOOTSTRAP_ADMIN_USERNAME=admin ^
-e KC_BOOTSTRAP_ADMIN_PASSWORD=admin ^
-e KC_HTTP_ENABLED=true ^
-e KC_PROXY=edge ^
-e KC_HOSTNAME_STRICT=false ^
quay.io/keycloak/keycloak:26.6.2 start-dev
Step 3 — Create a tunnel

Run:

cloudflared tunnel --url http://localhost:7080

Cloudflare will generate a public HTTPS URL that looks something like this:

https://random-name.trycloudflare.com

Example:

https://purple-waterfall-abc.trycloudflare.com
Step 4 — Configure the Meta redirect URI

In Meta for Developers, add this under:

Facebook Login → Settings → Valid OAuth Redirect URIs
https://purple-waterfall-abc.trycloudflare.com/realms/vilka/broker/facebook/endpoint
Step 5 — Configure the Keycloak hostname

Restart Keycloak and set the generated hostname:

docker run -d ^
-p 7080:8080 ^
-e KC_BOOTSTRAP_ADMIN_USERNAME=admin ^
-e KC_BOOTSTRAP_ADMIN_PASSWORD=admin ^
-e KC_HTTP_ENABLED=true ^
-e KC_PROXY=edge ^
-e KC_HOSTNAME=purple-waterfall-abc.trycloudflare.com ^
-e KC_HOSTNAME_STRICT=false ^
quay.io/keycloak/keycloak:26.6.2 start-dev
Why this works

Facebook only accepts HTTPS redirect URLs.

Cloudflare Tunnel gives you:

a public HTTPS domain
a valid SSL certificate
a secure tunnel to your localhost instance
no need to configure nginx or local SSL manually

So from Meta’s perspective, it’s talking to a real HTTPS website, even though Keycloak is still running locally on your machine.

Why the terminal window stays open

This command:

cloudflared tunnel --url http://localhost:7080

starts a live tunnel process.

If you close the terminal window, the tunnel immediately stops and the HTTPS URL stops working too.

That’s why the terminal looks “stuck” — it’s actively maintaining the tunnel connection.

You have a few ways to handle this.

Option 1 — Keep the window open

The simplest approach is to just leave the PowerShell window running while testing Facebook Login.

Option 2 — Use separate terminal tabs

A common setup is:

one terminal for Keycloak
one for cloudflared
one for React or Spring Boot

This is probably the most typical development workflow.

Option 3 — Run cloudflared in the background

You can also launch it as a detached background process:

Start-Process cloudflared -ArgumentList 'tunnel --url http://localhost:7080'

To stop it later:

taskkill /IM cloudflared.exe /F
One important limitation

Every time you restart a quick tunnel, Cloudflare generates a brand-new URL like:

https://new-random-name.trycloudflare.com

That means you’ll need to update:

the Meta OAuth Redirect URI
KC_HOSTNAME

every single time.

That’s the biggest downside of quick tunnels.

If you want stable URLs, you’ll need a permanent Cloudflare Tunnel or your own domain.

Better long-term solutions
Option 1 — Use a permanent domain (best overall solution)

Buy a low-cost domain from somewhere like:

Namecheap
Cloudflare

Example:

dev-vilka.com

Then create a stable subdomain like:

auth.dev-vilka.com

Point it to:

your VPS
a Cloudflare Tunnel
or even your local machine

Now your Meta redirect URI never changes:

https://auth.dev-vilka.com/realms/vilka/broker/facebook/endpoint

This is how most production-style OAuth development setups work.

Option 2 — Permanent Cloudflare Tunnel

Instead of quick tunnels, Cloudflare also supports named persistent tunnels.

That gives you a permanent URL like:

https://auth.yourdomain.com

Benefits:

stable URL
no repeated Meta updates
no repeated KC_HOSTNAME changes

Architecture:

Meta OAuth
   ↓
https://auth.yourdomain.com
   ↓
Cloudflare Tunnel
   ↓
localhost:7080
Option 3 — Deploy Keycloak once

Another simple approach is to deploy Keycloak to a hosted platform like:

Railway
Render
Fly.io

Then you can always use a stable HTTPS URL such as:

https://keycloak-yourapp.up.railway.app

For many developers, this ends up being the easiest long-term setup.

What most developers actually do

For OAuth providers like:

Facebook
Google
Apple

developers usually follow this pattern:

Early development

Use temporary tunnels for quick local testing.

Serious development

Move the auth server to:

a VPS
cloud hosting
or a permanent domain

because OAuth providers generally expect stable HTTPS origins.