OpenAPI
=======

This document is an introduction to discussons around how / why to consider an OpenAPI offering with Micro.

Background
----------

A PaaS for MicroServices is a useful thing, and Micro has certainly removed some of the regular hurdles which make this difficult. However, the journey doesn't end there - more often than not these services need to be made available to users over the internet. As we have seen with Proto and GRPC, having an expressive language to define these interfaces is extremely useful. Done well this can reduce the client-side burden considerably by allowing the use of code-generation and automatic documentation.

As much as we would prefer it if everyone could just "man up and use GRPC", this is not a realistic expectation if the desire is to minimise the burden of integration and encourage adoption.

OpenAPI (formerly "Swagger")
----------------------------

"OpenAPI" is Swagger v3. For all intents an purposes they are the same thing, but these days there is no reason to implement anything new as Swagger (v2). It is a YAML based DSL which describes every aspect of an API:

* Addresses
* Protocols (HTTP / HTTPs)
* Authentication requirements
* Security scopes (OAuth)
* Endpoints
* Methods (PUT / GET / POST / PATCH / DELETE etc)
* Query & path parameters
* Request payloads
* Response payloads
* Encoding (JSON, YAML, XML etc)
* Comments and descriptions for each

As you can imagine, a very expressive API definition can be achieved. Subsequently an extensive amount of code-generation can be performed, rendering pretty much everything apart from the actual business logic.

Potential ways to integrate with Micro
--------------------------------------

There are a number of ways in which this could be done, but I've outlined 3 here which represent a range of offerings.

### Auto-generate OpenAPI definitions from Protos with an in-house generator

It would be possible to generate a very basic OpenAPI definition from proto files, which would achieve the bare minimum really.

* In practise this would result in an API where every call was a POST with a JSON payload.
* As restrictive as this sounds, it _would_ still allow people to auto-generate a client.
* We'd even be able to inject the hostnames for the API endpoints for people who were using the Micro platform.
* More advanced features (such as auth and security scopes) may prove to be a challenge.

### Auto-generate OpenAPI definitions from Protos with GRPC-Gateway

The [GRPC-Gateway project](https://github.com/grpc-ecosystem/grpc-gateway) offers a generator which should be able to produce an OpenAPI schema based on an exiting proto. This is obviously going to be the easiest approach, but has some shortcomings:

* Protos require specific annotations in order to generate anything even vaguely resembling an expressive API document. This can be taken to an almost absurd conclusion which leaves your protos looking pretty weird.
* This would require us to run the GRPC Gateway

### Allow ingress to users own "mapping API" services

For users who want complete control over their public-facing APIs we _could_ consider offering ingress routes for them (based on *.their.domain for example). SSL headaches aside, this approach is worth discussing as it is the only way to _really_ provide the full power of OpenAPI.

* Users can compose their own OpenAPI schema definitions, and use something like [OAPI-Codegen](https://github.com/deepmap/oapi-codegen) to build an entire API service. This could be assembed from Micro libraries, and really ends with a generated set of clients for Micro services. OAPI-Codegen handles REST queries, routing, middleware, auth, payload validation... the works really (good project, and one that I've personally contributed several PRs to over the years).
* This API service would need to map these REST requests to GRPC requests, then map the responses back before returning them with the appropriate response codes.
* In this way the user is fully in control of the API behaviour, but again we'd have gone quite far outside the box of offering Micro products.
* Also by this stage the convenience is questionable, and arguably anyone trying to offer an API with this degree of customisation is probably not our target audience.


Difficulties
------------

* Some of the data models supported by OpenAPI do not map well to GoLang. Types such as "object" can only ever be represented as an interface{}, and constraints such as "oneOf" or "anyOf" again do not work well with strict static typing. The only real approach to this can be documentation along the lines of "The features of GoLang make supporting this difficult, so we suggest modeling your schemas in an alternative way..."
