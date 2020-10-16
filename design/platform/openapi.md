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


Difficulties:
-------------
* Some of the data models supported by OpenAPI do not map well to GoLang. Types such as "object" can only ever be represented as an interface{}, and constraints such as "oneOf" or "anyOf" again do not work well with strict static typing. The only real approach to this can be documentation along the lines of "The features of GoLang make supporting this difficult, so we suggest modeling your schemas in an alternative way..."
