# Platform

The platform is a cloud platform for Micro services development.

## Overview

The Micro platform is focused on being the simplest way to build, share and collaborate on microservices. This started 
as a framework and now moves into a platform which continues to remove friction from the path of the developer. 
We'll keep climbing the stack to continue on this journey. From platform to services and beyond.

## Features

The features which will be included in the platform

- **Managed services** - Services will run in the cloud and managed by default.
- **Auto configuration** - Services will be auto configured with the required registry, broker, etc
- **Simplified debugging** - Stats, logs and tracing will be available in an easy and clean way for every service
- **Connect from anywhere** - Ability to connect and leverage the platform and services from anywhere

## MVP

Our MVP goal is to deliver Micro as a Service. We're taking the open source micro toolkit 
and running it as a service in the cloud. The end state should be a runtime which offers an API, web dashboard, 
CLI, slack bot and service proxy all hosted so that developers don't have to host it themselves. We'll 
provide interconnectivity via a transport, broker and service registry which can be accessed by running 
local `micro` instance which connects to the service in the cloud. 

Hosting will be provided by global deployments in every region using the `micro run` command.

We'll be dogfooding these efforts on an ongoing basis by writing microservices which run on the platform 
and are entirely open source for others to consume. Ultimately our goal is to see the network with 
base level services that don't have to be written or run by anyone else. 

We want to introduce this by end of Q1 2020 as an invite-only beta mode for our community to test. Most of this 
software will be entirely open source and available to run so anyone can likely connect but we'll 
look to build identity and api tokens in for access control.

Ultimately we'll provide a **M3O** (pronounced ehm-three-oh) as a global platform for Micro services.

## Goals

We have the following high level goals:

- [x] Bootstrap Micro OSS in the cloud aka Micro as a service
- [x] Dogfood by writing services or a complete end-to-end service
- [x] Consolidate learnings into roadmap and bug-feature list
- [x] Develop the network to allow connectivity from anywhere
- [x] Enable sharing of services and discovery
- [x] Hosting services on the platform
- [x] Auto configuration of services
- [x] Observability and debugging; stats, logs, trace
- [x] Tokenized access control
- [x] Invite-only service for the community using Github
- [x] Onboard users and iterate based on the feedback

## Beyond MVP

Beyond the MVP we want to move into focusing on extending the platform, including team based collaboration and private hosting.

- [x] Privately hosted Micro platform
- [ ] Team based features and access control
- [ ] Paid personal and team subscription
- [ ] Value add services beyond the runtime

## Micro as a service

Ultimately we want to deliver micro as a service as a shared platform for microservice development. Anything a company 
ends up building themselves should be available as a shared runtime for all developers everywhere in the world. This should 
not exclude those who want to self host and access the system as if it were part of their own.

### Summarised view

- [x] Micro Platform - a shared platform for service development
- [ ] Multi-Cloud {AWS, Azure, GCP, DO} - highly available and invisible to the user
- [ ] Multi-Client {Go, Python, Ruby, Javascript, Rust} - empower developers in any language
- [ ] Services {Platform, Utility, Orchestration, Web} - base level services delivered by micro itself

### Further breakdown

- [x] **Platform** {Micro Server, Proxy, Runtime, Sync, Network}
  * Evolve the platform => build, run, manage
  * Establish go-micro as micro server
  * Enable connectivity beyond the cloud
  * Continue development of core services
- [x] **Namespaces**
  * Namespacing services for separate teams
  * Owner/group baked in at runtime
  * Controlled ability to expose on per service basis
- [ ] **Billing**
  * Usage tracking
  * Accounts and payments
  * Subscriptions
- [ ] **Debugging** {Logging, tracing, metrics}
  * Build in observability features
  * Runtime stats; cpu, mem, threads, requests, errors
  * Logs; historic logs for a finite time
  * Distributed tracing for the call graph
- [ ] **Access Control**
  * Team based access control
  * User / Service / Admin scopes
- [ ] **Routing**
  * Label based routing (Micro-X Headers route via metadata)
  * URL Versioning routes to service versions
  * Per web app sub domains
- [ ] **Monitoring / Alerting**
  * Monitoring of services
  * Ability to define healthchecks
  * Events, emails and paging where necessary
  * Scheduling of on-call (stretch goal 6-12 months)
- [x] **Commands**
  * Auto generate CLI commands based on available services
- [ ] **Services** {Platform, Utility, Orchestration, Web}
  * Continue key service development to 30+ services
- [ ] **Clients** {Go, Python, Ruby, Javascript}
  * Enable multi-language through micro server + grpc
- [ ] **Multi-Cloud**
  * Run on DO, AWS, Azure, GCP
- [ ] **Multi-Network** {Micro Services Network}
  * Global services network
  * Shared system for building microservices
  * Micro Servers connecting to form network
- [ ] **Multi-Lang Apps**
  * Add additional languages (rust, java, etc}
  * Concept of 'cells' to encapsulate code in containers
  * Use 'micro service' command to turn into a service
- [ ] **Mu spec**
  * Flesh out the mu spec for single file definition
  * Move beyond proto and go
  * DSL for orchestration

