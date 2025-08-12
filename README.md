# Patch for getting n8n to use LLM without Timeouts

This comes WITHOUT WARRANTY to work nor to not delete your data!
You have been warned!

I'm not involved in n8n, and I'm not an expert in TypeScript whatsoever - so use at your own risk!

# Prereqs

n8n has two timeouts that don't work with long running LLMs.
First, node.js introduced an http timeout value, which is actually (AUG 25) not set and there fore times out at 5 minutes.
http-Requests taking longer as this will time out (so for the frontend of n8n).

The second one is with undici used for the connection between n8n and the LLM.

Both need to be enhanced.

# Docker image

Enhance the docker image by undici - see the Dockerfile in the repo here.
It will just install unidici in the image.
Make sure to use that image build further on!

# Patch

Use the patch given in this repo and mount it into a running container - that also might be done just read only.

# Change / Add your docker environment variables

Add the following values to your docker environment variables, either in a docker compose or your .env file.

Make sure to replace the /path/to/your/mount with the path where you mounted the patch at.

```
    - NODE_OPTIONS=--require /path/to/your/mount/patch-http-timeouts.js
    - N8N_HTTP_REQUEST_TIMEOUT=0       # 0 = disable per-request timeout
    - N8N_HTTP_HEADERS_TIMEOUT=120000  # 2 minutes; must be > keep-alive
    - N8N_HTTP_KEEPALIVE_TIMEOUT=65000 # 65 seconds
    # Outbound fetch/undici timeouts
    - FETCH_HEADERS_TIMEOUT=1800000   # time allowed to receive response headers
    - FETCH_BODY_TIMEOUT=12000000     # total time allowed to receive the body/stream
    - FETCH_CONNECT_TIMEOUT=600000    # total time for waiting for a connect
    - FETCH_KEEPALIVE_TIMEOUT=65000
```

You may also push that file into your docker image - I didn't as I would like to be able to configure / change things later without beeing forced to build the image again.

---

Thats it - now your long running LLM-Calls should go thorugh.
