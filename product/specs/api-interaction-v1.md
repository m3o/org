# API Interaction V1

How do users interact with the API

## Requirements
- Protect usage of the API through standard means (API key) 
- Track invocations of endpoints (count) per user
- Invoke the API using standard tools (CURL, http libraries, etc)

### Out of scope
- Revoke api key
- Metering and quotas (we'll track the number of invocations but not put in quotas) 

## User flow
1. sign up for m3o account
2. hit service to generate an API key. Probably from a dashboard page. 
3. pass api key on api calls with "Authorization: Bearer <TOK>”.   

## Backend flow

### /v1/generate
The generate endpoint mints a new auth account and api key for the caller. New account because we don't support multiple authentication secrets for a single account. It's probably a nicer way to enforce separation anyway. 

When a request to `https://superapi.m3o.dev/v1/location/track` hits the API it does the following
1. existing micro `api` service will forward the request to `superapi` namespace
2. request is forwarded to new `v1api` service (naming is hard) as the entrypoint. This is what manages things like tracking the number of invocations of the endpoint, checking the token hasn't been revoked, etc. It's doing a lot of the work of a fully fledged API gateway basically. Eventually we'll want to push the logic up in to the actual micro API service but for the sake of v1 we'll stick to a new service
3. Once satisfied that the request is good, the request is forwarded on to the appropriate service

## Implementation
API keys are opaque tokens rather than JWTs. This is better because 
- they are shorter and easier to copy/paste
- they hold no data so people can't inspect them and try to do nefarious things
- can't be abused to work against the normal API and m3o platform

When the request hits the v1api we do a lookup and essentially swap the opaque token for a JWT which is passed through for the rest of the request's lifespan. The JWT is stored in the DB for quick retrieval but is deliberately short lived (1 hour) and only refreshed on demand to limit the blast radius if the DB was ever hacked. i.e. in case of breach all JWTs would be exposed but only a proportion would still be valid and their lifetime would be less than 1 hour. We can change this lifetime to increase security as required. 

All tokens have `superapi` as the issuer. Using superapi.m3o.dev means we don’t need to specify `Micro-Namespace` as `superapi`. Gives us time to work out how to change the routing (if at all) so that we can run things in other namespaces and route automatically without having to specify that header. 

Using a new namespace rather than `micro` means we don't need to lock down the core services using scopes yet which is fiddly.

New service `v1api` that handles api gateway things. Has the following endpoints
- `/v1api/generate` - generate an api key with appropriate scopes (scopes will probably be hardcoded for now)
- `/v1api/*` - catch all that handles all inbound and forwards to appropriate service 

### Notes for service implementation
Individual services need to check the scopes on the JWT to decide whether the request is auth’ed.

Individual services need to implement multi tenancy to ensure user data is appropriately protected although in future perhaps we can automatically do this by passing the context to store calls


