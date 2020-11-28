# Organisation

The Micro company handbook, product roadmap, team and contribution guide.

# Overview 

Micro is the fastest way to build, share and collaborate on services in the Cloud and beyond. Our goal is to continue on this mission in an open and collaborative way, both as a team and community. Up until now most of our ideas and development have revolved around discussions in [Slack](https://slack.m3o.com).

This repo serves as an open forum for collaboration, design docs and the roadmap.

## Contents

- [company](company) - collaboration, culture, growth, rules of engagement, vision
- [design](design) - where the design docs live
- [glossary](glossary.md) - a glossary of terms and explanation of brand, company, project, product
- [ideas](ideas) - a scratch space for new ideas and things in flux
- [issues](https://github.com/m3o/development/issues) - where we track work
- [office hours](#office-hours) - times we are available to the community
- [philosophy](#philosophy) - how we design and work
- [product](product) - product and planning related docs
- [release](company/release-process.md) - our release process
- [roadmap](roadmap) - where the product roadmap lives
- [style guide](#style-guide) - coding style guide

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

## Contributing

- Open an [Issue](https://github.com/m3o/development/issues) to start a discussion with the community
- Open a [Pull Request](https://github.com/m3o/development/pulls) to add a design doc where you've already got strong ideas
- Help shape the roadmap for Micro and be a part of the microservice movement
- Take any real time conversations to the [#m3o-dev](https://micro-services.slack.com/messages/CJ544CH8W/) channel on Slack

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

Office hours are times we are available to the community. This is an experimental feature we're testing. 

Fridays 2-5pm GMT we'll have an open call anyone can join to chat, ask questions, etc on [Discord](https://discord.gg/hbmJEct).
