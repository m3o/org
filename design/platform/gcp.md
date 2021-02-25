Google Cloud Platform
=====================

The following rant is an attempt to capture some thoughts on the subject of public clouds, platform deployment, and service deployment.


Motivations
-----------

- Sick of 2nd-rate clouds, their lack of features, poor stability
- About to get some "rolling credits" for GCP
- Drive to switch to managed services for Micro platform (in preference to self-hosting)
- Eventually reduce developer time spent managing clouds, reduce time-to-market for new features, reduce support costs, and increase stability

I'm a bit worried that the focus is again on cloud-hosting costs. Consider the heavy opportunity cost already incurred by jumping from cloud-to-cloud to claim credits, and the huge diversion in developer effort with each migration to date. Eventually this effort has very little benefit for the end-user, and could probably have been better spent developing features and services. Always focusing purely on cost usually turns out to be a short-sighted decision that results in the engineers responsible for maintaing the platform taking on extra stress, which is something difficult to quantify in financial terms.

Obviously switching clouds again should only be considered if the motivations take these points into consideration.

It would probably make sense to break this into two distinct phases: switching to GCP first, then swapping out services second.

I'll also go into some other aspects of a standard aproach to managing these kind of deployments.


A "classic" approach (pre-building infrastructure before deploying Micro)
-------------------------------------------------------------------------

In this approach we provide the infrastructure for Micro to run on top of, plugging in as we currently do.

- Pre-build infrastructure from code
- Use as many iaas / saas offerings as possible from GCP (databases, message buses, monitoring etc)
- Any self-hosted components would then be deployed to the new K8S cluster
- Deploy Micro from pre-build container images


### Infra-as-code

I understand that there have been some issues in the past with Terraform, but in my opinion it makes no sense to use anything other than Terraform to provision infrastructure in public clouds (as long as we're assuming the pre-created infrastructure approach).

Terraform has been the go-to tool for many years now, and is constantly in active development. This ensures that new services / features from your cloud provider will be supported as quickly as possible, and that bugs are discovered / fixed rapidly. Having said that, a workflow needs to be understood and established. I'd propose something along these lines:

- Make use of Terraform Cloud's free tier
- Configure a workspace for each Micro environment (dev / prod / whatever)
- Configure each workspace to watch for changes to the TF code in the github.com/micro/micro repo
- Any PR will automatically trigger a "plan", which is included as a GitHub check
- An approach of "only applying from master" is sensible if we can manage. This means that once a PR is merged, TF Cloud will run another plan. If that was successful then we can run the apply
- All of these operations are run through a web interface which also offers secrets management (so we don't have to check secrets into github or rely on some crufty encryption scheme)

There is no denying that Terraform is the standard way of managing infrastructure-as-code, and for many years it has been quite a reasonable expectation that any new devops hire will already have been using Terraform in previous roles, and can hit the ground running. Well-structured Terraform code also serves as the best possible form of documentation for your infrastructure. I know nothing about Pulumi, but getting to the point where the world has a standard way of doing something which used to be extremely difficult has been a game-changer.


### Micro services

Sticking with the theme of standardisation, Helm is still the obvious standard for managing Kubernetes deployments, and ever since version 3 has been an absolute pleasure to work with. Slightly verbose yes, but flexible and worth it in the long run also yes.

Helm has already made it extremely easy for us to deploy open-source software solutions off-the-shelf from the public chart repository, and we should endeavour to make the experience of deploying Micro identical to that. This not only serves us but also any of our users who decide to self-host the Micro platform.


### GCP services

Once this has stabilised we can look at migrating to cloud-native solutions. At first glance through https://github.com/m3o/platform these candidates spring to mind:

- Cockroach _could_ be swapped out for Google Spanner, but the pricing model of Spanner involves renting and instance-based cluster 24/7. I would question the need for this type of database altogether. Why do we need Cockroach over something like plain old Postgres? PG can scale a long way, is infinitely more widely understood, and is offered as a basic DB service by nearly any cloud provider. If vertical scaling becomes an issue then horizontal sharding is an obvious choice for us, given that we mostly do not want to share data between customer accounts.
- Swapping Nats for Pub/Sub makes more sense, as the pub/sub pricing model is truly pay-per-volume. In my experience pub/sub was an excellent service and is one of the best features available on GCP.
- Making use of GCPs identity-aware proxy may prove to be worthwhile if we're sick of the existing VPN infrastructure. Given that it integrates with Google's auth (which we're already using for groupware and the cloud console) this becomes a truly single-signon experience.
- Logging is another area where we stand to shed some self-hosted tooling. StackDriver is an off-the-shelf offering with GKE, where with the tick of a single box we will suddenly have logs from every container routed to an infinitely scalable aggregated location. Compared to AWS CloudWatch this is actually a very usable service where structured logs are fully indexed and able to be filtered. Combine this with the quality of their API and CLI tools, and we start to have a basis to not only make us of our own logs, but to programatically make them available to customers on a per-namespace basis.
- Monitoring is another aspect where we stand to get a lot with very little effort. All GCP services (including GKE and nodes/namespaces/deployments/containers) are automatically monitorable with StackDriver. The StackDriver monitoring UI isn't up there with Grafana, but Grafana does have a StackDriver plugin which we could use to build decent dashboards. On top of this, the GCP client library for GoLang automatically supports StackDriver tracing for most of their services, which is something quite beautiful to behold once you've seen it in action.
- Tracing is the next thing. StackDriver tracing is again very usable and would be my preferred option over self-hosting Jaeger. This becomes obvious once you start tracing at scale, as it very quickly becomes a big-data problem.


An alternative approach (give Micro the keys)
---------------------------------------------

Worth mentioning (so it can at least be said that it was considered) is the concept of just giving Micro some GCP credentials and letting it run wild. This "turnkey" approach might sound initially appealing:
- No need for Terraform
- Simple
- Get going quickly

But these benefits would only be seen by our users who have decided to self-host. For us (Micro) it means more:
- This would take a considerable amount of time to implement
- It would need to be constantly maintained to keep parity with GCP features and mods
- Any new "devops" hires would now need to be GoLang devs (not impossible, but devops types are already difficult enough to find)
- Even more cruft in the Micro codebase

Unless this was a clear path to revenue I would steer clear (at least for now), and focus on services which bring benefit to users of the Micro Platform.
