# Onboarding

This is onboarding for anyone working at Micro.

## Overview

We need a set of steps and procedures to bring someone upto speed with micro. This is for anyone who joins the team and needs to hit the ground running. Part of that is getting access to tools, setting up an environment, but also getting to know team members and learning about the bigger picture.

## Getting Started

- [Add to accounts](#add-accounts)
- [Schedule calls](#schedule-calls)
- [Setup your environment](#setup-environment)
- [Create a service](#create-a-new-service)
- [Read the docs](#read-the-docs)
- [Walk through examples](#walk-through-examples)
- [Build some services](#build-some-services)
- [Deploy to the platform](#deploy-to-the-platform)

## Add Accounts

Team day 1 onboarding requirements:

- Add to Gmail
- Add to Slack
- Add to Cloud {Scaleway, DO, GCP, ...}
- Add to GitHub
- Add to DockerHub
- Add to 1Password
- Add to Stripe
- Add admin accounts for dev and platform tiers of m3o
- Add to Cloudflare
- Add to Linear

## Schedule Calls

Reach out to each team member and schedule 1:1 calls to get to know colleagues. Add to regular team calls.

## Setup Environment

Download and install Go from [https://golang.org/dl/](https://golang.org/dl/)

If you're new to Go walk through the tour [https://tour.golang.org/welcome/1](https://tour.golang.org/welcome/1)

Ensure go modules are enabled and your PATH is set:

```bash
export GO111MODULE=on
export PATH=$PATH:$(go env GOPATH)/bin
```

### Install Micro

```bash
go get github.com/micro/go-micro/v2
go get github.com/micro/micro/v2
```

Also, install protobuf and the relevant generators:

```bash
brew install protobuf
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
go get -u github.com/micro/micro/v2/cmd/protoc-gen-micro
```

## Create a new service

Create a new service

```bash
micro new github.com/micro/mynewservice
```

Compile the proto file

```bash
cd github.com/micro/mynewservice/
protoc --proto_path=. --go_out=. --micro_out=. proto/mynewservice/mynewservice.proto
```

Run the service

```bash
cd github.com/micro/mynewservice
go run main.go
```

Lookup the service

```bash
micro list services
```

Get the service

```bash
micro get service go.micro.srv.mynewservice
```

Call the service

```bash
micro call go.micro.srv.mynewservice Mynewservice.Call '{"name": "John"}'
```

Start the API

```bash
micro api --namespace=go.micro.srv
```

Call via the api

```bash
curl -XPOST -d '{"name": "John"}' http://localhost:8080/mynewservice/call
```

Start the web app

```bash
micro web
```

View at http://localhost:8082

## Read the docs

Skim through the docs at [https://micro.mu/docs/](https://micro.mu/docs/) to get an overview.

And the design docs in this repo. Make a note of missing or stale content.

## Walk through examples

Walk through the examples at [https://github.com/micro/examples](https://github.com/micro/examples)

- Start with the basic [helloworld](https://github.com/micro/examples/blob/master/helloworld/main.go)
- See the end to end app example with the [greeter](https://github.com/micro/examples/tree/master/greeter)
- How to do bidirectional [streaming](https://github.com/micro/examples/tree/master/stream)
- Simple [pubsub](https://github.com/micro/examples/tree/master/pubsub) via the go-micro/broker

## Build some services

Write some services using [https://github.com/micro/go-micro](https://github.com/micro/go-micro) to explore the framework.

Example services can be found here [https://github.com/micro/services](https://github.com/micro/services)
 
## Deploy to the platform

Get credentials from the team to deploy to the micro platform
