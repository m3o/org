# Software Development Lifecycle

## Tools
We use https://linear.app/ to manage our day to day work.

We use https://github.com projects to manage a [public roadmap](https://github.com/m3o/dev/projects/11).

## Process
We loosely follow the Scrum methodology and pick and choose ceremonies as appropriate. As we grow in size we will no doubt adopt more and apply them more rigorously. 

### Standups
We don't do daily standups, rather, every morning at 10am we post our standup update in to the #team-important channel in Slack. We post what we did yesterday, what we plan to do today, and any blockers we may have. 

### Sprint cadence
We run 2 week sprints. Work is identified for the sprint at the beginning of the period to fill the two weeks but not overspill. If a piece of work is scoped to last longer than a sprint then it is too big and should be scoped in to smaller chunks. If we run out of work then we can either pull in additional work that will fit in to the remaining time left in the sprint or we can work on bug fixes or "20% time" type work.

We will generally try to organise work for a sprint to be around a theme to help give a bit more cohesion to releases. e.g. a sprint based around debugging may be focussed around features that help with debugging like tracing, profiling, logs.

### Planning
On the Monday at the start of every sprint we'll have a planning session to go pull in all the issues for the upcoming sprint. Ideally any work that is planned for a sprint should already have a full spec of requirements and/or acceptance criteria such that any developer can pick up that ticket and understand what needs to be delivered; if it does not then it is not eligible to be worked on by a developer. Tickets that need further elaboration are labelled with `spec` in Linear. 

At the start of a sprint all tickets in that sprint will be in the `To do` column of the board. 

The `backlog` in Linear contains all the work / issues that we want to look at near term. Anything longer term should be pushed out to the public roadmap. Tickets in Linear that are not updated after 3 months are automatically archived; if it's important it will bubble up again or would have been fixed before that.

### Working on tickets
When a developer picks up a ticket then should assign themselves to the ticket and move to `In progress`. Tickets labelled with `design` require a design doc to be shared with the rest of the team before implementation. This is to save any wasted effort by implementing the wrong thing. A design doc should clearly explain the proposed approach/architecture to solve the problem and should be added as a PR to https://github.com/m3o/dev/tree/master/planning/specs to allow easy commenting. It may be text based, include diagrams etc. Once the team are happy and in agreement, the PR can be merged and work can commence on the code.

Code is reviewed and merged in the normal way via github PRs. If something is taking a while to be reviewed you can move the ticket to `In Review` in Linear. Once merged the ticket can be moved to the `Done` column.

Try to avoid having too many tickets in flight at once, chase up any approvals/reviews you may need if you're being blocked.

### Deploying
Anything which can be deployed straight away, typically changes to backend services or web properties should be done as soon as possible after merging the code. The developer who made the change should really be the one to deploy to encourage ownership of our work. Things which can't be deployed straight away should mostly be changes to the micro cli or oss code and we'll aim to cut new releases every 2 weeks (e.g at the end of the sprint)

