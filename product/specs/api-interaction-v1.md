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
2. hit service to generate an API key (JWT). Probably from a dashboard page. 
3. pass api key on api calls with "Authorization: Bearer <TOK>”.   

## Backend flow
When a request to `https://api.m3o.dev/v1/location/track` hits the API it does the following
1. existing micro `api` service will forward the request to `micro` namespace
2. request is forwarded to new `v1api` service (naming is hard) as the entrypoint. This is what manages things like tracking the number of invocations of the endpoint, checking the token hasn't been revoked, etc. It's doing a lot of the work of a fully fledged API gateway basically. Eventually we'll want to push the logic up in to the actual micro API service but for the sake of v1 we'll stick to a new service
3. Once satisfied that the request is good, the request is forwarded on to the appropriate service

## Implementation
All tokens have `micro` as the issuer so everything is routed to `micro` namespace on the platform. This means we don’t need to specify `Micro-Namespace` as `micro` is the default in the absence of any specified namespace. Gives us time to work out how to change the routing (if at all) so that we can run things in other namespaces and route automatically without having to specify that header. 

It does mean we need to lock down the core services appropriately and make them check scopes not just that the issuer is `micro` as we do currently. Things to highlight:
- Users can create auth accounts using `auth generate`. The current mechanism means that they can only create accounts where the issuer is their namespace. If we give people accounts in the micro namespace we need to make sure that they can only create accounts with the appropriate privilege level e.g. a user shouldn't be able to create an account with escalated privileges higher than their own.    

New service `v1api` that handles api gateway things. Has the following endpoints
- `/generate` - generate an api key with appropriate scopes (scopes will probably be hardcoded for now)
- `/*` - can we have a catch all endpoint that catches all other calls to `/v1/*`? If we can't then we'll need to push this stuff up in to `api` service 

### Notes for service implementation
Individual services need to check the scopes on the JWT to decide whether the request is auth’ed.

Individual services need to implement multi tenancy to ensure user data is appropriately protected although in future perhaps we can automatically do this by passing the context to store calls


