# Services

Micro Services are services offered on the M3O platform

## Overview

M3O is a cloud native platform for Micro services development. It is host to services offered by Micro to the world. 
Services are Go Micro based services serving a specific domain boundary via an RPC interface. Services provide 
some value but also act as building blocks for other services to be built on.

Services can extend beyond RPC to also provide an API, Web or Mobile apps.

## Server

Services provided by the core runtime

- [x] **Auth** - Authentication and Authorization of users and services
- [x] **Broker** - Async publish and subscribe messaging
- [x] **Config** - Dynamic application level configuration (TODO: secrets)
- [x] **Debug** - Debugging service info; logs, stats, tracing
- [x] **Events** - Event aggregation and storage
- [x] **Network** - Multi-environment service networking and latency based routing
- [x] **Registry** - Service registry, endpoint and metadata explorer
- [x] **Runtime** - Service provisioning and lifecycle management
- [x] **Store** - Distributed key value storage service
- [ ] **Sync** - Distributed synchronisation; locking, leadership election, maps, events, cron, workers
- [ ] **Monitor** - Healthchecking, services monitoring and user defined checks
- [ ] **Identity** - Distributed CA and strong identity
- [ ] **Token** - Random opaque token generation for one time usage of APIs

## Clients

External entrypoints also within the micro server

- [x] **API** - API gateway for http/json api construction
- [x] **CLI** - Terminal command line interface
- [x] **Proxy** - Service to service and external proxying to services
- [ ] **Chat** - Slackbot for ChatOps
- [ ] **Web** - Web dashboard and proxy for web apps as microservices
- [ ] **Voice** - Ok Google, Siri, integration for voice commands
- [ ] **Mobile** - On mobile using Flutter, equivalent of Web

## Services

A non-comprehensive list of high-level services:

- Alerting
- Audio
- Backups
- Broadcast
- Checkout
- Customer
- Database
- Email
- Event
- Food (markets)
- Fraud/Risk
- Gamify (scores / leaderboard)
- geocoding
- i18n
- Image
- Location
- Maps
- Message
- Monitoring
- Notes / todo / lists
- Notification
- Payment
- Personalisation
- Place
- Popup (locations)
- Review (hyper local ephemeral reviews)
- Reward (points / tokens)
- Search
- Secrets
- Status
- Stream
- Support
- Timeline
- translation
- User
- Video
- Voice
  
Testing & security services:

- **Fuzzing** - Sends random garbage to rpc endpoints
- **Chaos** - Attempts to destroy the runtime by stopping or killing things
- **End to End (E2E)** - Allows end-to-end flows for real world product tests

