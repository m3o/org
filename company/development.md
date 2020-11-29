# Development

A simple one page description of development at Micro

# Overview 

Micro is the fastest way to build, share and collaborate on services in the Cloud and beyond. Our goal is to continue on this mission in an open and collaborative way, both as a team and community. Up until now most of our ideas and development have revolved around discussions in [Slack](https://slack.m3o.com).

## Philosophy

Micro is the simplest way to build microservices. Our goal is to make developers incredibly productive, 
remove friction from their workflow and abstract away the complexities of distributed systems and cloud-native technology. 

Here's how we approach taking on new problems.

1. Engage the community about the problem and how we need to solve it.
	- Discussions lead to clarity in thinking
	- We can work through ideas before actually spending significant time writing code
	- We can understand if we're even solving a problem
1. Define the overall scope of the project and name it.
	-  e.g Auth, Config, Debug. This is the domain boundary
	- Network deals with inter-service communication
	- Config deals with dynamic configuration
	- API is an API gateway for HTTP requests
2. Start with a Go interface, this is always our starting point, we want to solve for the developer in Go first. 
	- Start with the high level interface that will be used. Implement it and make it pluggable
3. Ship quickly and iterate, test the ideas with the community.
	- Go Micro was being used 2 weeks after the first line of code was written..
4. Encapsulate as a command/service in Micro so that it solves the problem across all languages
	- Config starts as an interface/importable package for user usage
	- Implemented as the config service built on the interface
	- Embedded as a command `micro service config`
5. Everything that we do focuses on simplifying the experience for the developer
	- Provide a zero dependency default experience while being pluggable
	- Abstract away cloud-native and distributed systems. Operations is a separate concern
6. Build on our own foundations rather than those of others.
	- CNCF projects are complex, fast moving, breaking and we have no control over their long term goals
	- We want to own the foundations so we can build on them and focus on making developers productive
	- Where other tools solve complex problems we offload with plugins or abstract them away

## Style Guide

The coding style guide is fairly straightforward.

- **KISS:** - Don't use complex algorithms where a for-loop would do. Just keep it simple. We'll fix perf later. 
- **Brevity:** - Don't use long variable names where a comment would suffice. Do the minimal work.
- **Consistency:** - See the surrounding packages, variables and types as a guide for how you write code.

## Workflow

When you are contributing whether it be bugs or features you are the owner of that through to completion.

The workflow is as follows:

- Discuss: Talk about the the bug or feature in #development with others first
- Document: Create an issue and todo note to track starting and progressing the task
- Deliver: Create a PR for your bug or feature and engage at least 2 other team members to review
- Verify: You should be testing throughout but once delivered its your responsibility to verify full functionality

## Office Hours

Office hours are times we hold open calls to chat over voip. This is an experimental feature we're testing. 

Fridays 2-5pm GMT we'll have an open call anyone can join to chat, ask questions, etc on [Discord](https://discord.gg/hbmJEct).
