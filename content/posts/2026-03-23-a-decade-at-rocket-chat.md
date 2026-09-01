+++
title = "A Decade at Rocket.Chat: Looking Back on 10 Years"
date = 2026-03-23T00:00:00Z
layout = "post"
publish = true
comments = false
tags = []

+++

After 10 years at Rocket.Chat... I'm saying goodbye. I started contributing to Rocket.Chat after it appeared on Hacker News in 2015. Some of my first pull requests were fixing image stretching bugs, hubot integrations, jitsi integration and many others.  Then quickly became the Cloud/Deployment guy after being hired.  

When I joined my world was pretty small.  I hadn't been out of country much, and I only ever worked with people I had met in person.  Now i've traveled and worked with people from all around the world.

I want to look back on this journey. Not just the technical milestones, but how I grew as a technical professional, the team evolution, the org changes, the migrations that aged me, and the people who made it all worth it.

## 2015: A Community Contributor Fixing Bugs 

I discovered Rocket.Chat in 2015. **Bradley Hilton** and I had actually been building our own chat application. It started because he needed a staff chat for his Minecraft server and was tired of using Slack (and definitely didn't want to pay for it). We couldn't find anything off the shelf that fit the need.  So we started building our own in Golang. When I found Rocket.Chat it was almost exactly what we were trying to build. So instead of reinventing the wheel, I started using it, hit rough edges, and started fixing them. My first pull request on August 21st, 2015 was titled "Fix image stretching." The second one came the same day: "Fix search not coming up."

Within a week I was submitting PRs for pin/unpin functionality, configuring the site name via environment variables, adding `/invite` and `/leave` slash commands, and improving hubot-rocketchat. 33 PRs across 4 repos before the year was out. I wasn't an employee. I wasn't getting paid. I was just a guy who liked the product enough to make it better.

In October of 2015, Diego Sampaio built the first easy way to try Rocket.Chat, a proof-of-concept system brought online for **OSCON in Amsterdam**. You'd enter your email and a subdomain like `something.rocket.chat` and it would spin up a workspace. Built on Rackspace with Tutum, a single Mongo node, a single worker, and Hipache for load balancing. The address was `deploy.rocket.chat`. Simple. Scrappy. It worked.

The seeds of everything that would come later were planted right there.

By mid-2016 I was going deeper, I contributed the original **Jitsi video conferencing integration** in the hotel lobby across from **OSCON in Austin**.  We met Emil Ivov the founder of Jitsi earlier that day in the Atlassian office.  Got excited and started building.

## Fleetcommand and the Cloud Team (2016 - 2018)

In **September 2016**, we built the `rocketchat-server` snap, one of the very first snaps created. This was a direct result of interactions with Mark Shuttleworth from Canonical. A small but cool moment in our open-source story.

By **October 2016**, it was becoming clear that Rocket.Chat was going to receive its first round of funding (which officially came in **November 2016**). I started building **Fleetcommand**, the software that would manage trials, handle Stripe billing, and automate provisioning for our hosted customers. The original vision was a portal at `my.rocket.chat` for users to manage their workspaces. We explored many options like using Juju by Canonical(after earlier success with snaps), docker swarm, mesos, nomad and several other ideas.  But ultimately decided on kubernetes and utilizing its API and specifically the same go packages kubectl leveraged.

In **January 2017** we received our Google Cloud credits, and Fleetcommand went live around **March 4th, 2017**.

A fun story during this time: I had traveled to Brazil on February 23rd, 2017 for our first ever company summit. While there, we received an opportunity for a free booth at Google Next in San Francisco only a few days later, March 8th to 10th. So we flew from Brazil to San Francisco.  I was gone for 20 days.  My wife was not so sure about this new startup I had joined! 😂

![The Rocket.Chat team at Google Cloud Next 2017](/images/2017-03-08/google-next-rocketchat.jpeg)

**At the booth, we switched the deployment of workspaces from rackspace/tutum to run on Kubernetes on Google Cloud.** We were literally refining the process and deploying fixes live from the conference floor. We even rushed a proof of concept that automatically linked users to their Google accounts. Once it was finished, we actually then returned to Brazil and had our summit.

![Fleetcommand hosting page going live](/images/2017-03-08/google-next-fleetcommand-live.png)

### The First Great Migration

For a while we supported Google Cloud alongside the original Tutum deployment. But as the Rackspace credits ran out, we needed to migrate everything over. This was our **first zero-downtime migration** and we developed the pattern we'd use for every migration after.

The strategy: extend the existing `use1-db1` MongoDB replica set by standing up additional replicas within Google Cloud. Once the new replicas were healthy, switch the primary over. Spin up all new workspaces in Kubernetes on Google Cloud. Switch the DNS records. Completed **March 26th, 2017**.

At this point I was basically a solo operation as the "Cloud Architect." I was writing Fleetcommand, managing infrastructure, handling deployments, writing documentation. 93 PRs across 23 different repositories in 2017 alone. My fingerprint was across the entire ecosystem.

### Becoming a Team

Early **2018** after our summit is when the "cloud" team was born. Bradley, who had been present since the company was formed, joined the cloud team. He was initially focused on the Apps Engine, but his focus pivoted to needing a way to distribute apps, which led to the creation of the **Marketplace**. Its first commit landed on **July 14th, 2018**.

In **July 2018** we also made our first infrastructure-specific hire. The team grew from 2 to 3.

If you look at my GitHub history from the second half of 2018 there's a clear pattern: take a service, containerize it, add automated builds with Google Cloud Build, deploy it. On **July 31st** alone I submitted PRs adding a Dockerfile and auto-deploy to the marketplace-api *and* adding Prometheus to Fleetcommand. Three days later: "Release initial version" of cloud-portal. Then statuscentral, our status page which we'd built in about 2-3 days in Go inspired by another open-source status page, got Docker and Cloud Build (September 27th). That little project would keep evolving for the next seven years: maintenance windows, a CLI, snapshot ability, incident history, even a Twitter integration so customers could follow along during outages. Then the Push Gateway was rewritten from Javascript to Go and containerized (November 15th). The Javascript version was locking up under load. The process would stop responding and the only fix was killing the pod. The Go rewrite was a night and day difference. The memory footprint went from around a gigabyte down to ~100 megabytes and cpu usage went way down. The OmniChannel Gateway got a multi-stage Docker build. The Federation Hub got simplified and containerized. One by one, the cloud platform took shape.

By the end of 2018 we had the bones of the cloud platform. I had 142 PRs across 27 repos that year, and the operational burden of managing a growing hosted customer base was growing right alongside it.

## 2019 - 2020:The Great Migrations 

### 2019: Google Cloud to AWS

We operated on Google Cloud until those credits also began to diminish. Amazon approached us with AWS credits, so we kicked off our **second major migration**. This time the approach was a little bit more sophisticated. Like last time, we established a secure tunnel between the two environments so they could talk to each other, and extended MongoDB replica sets from Google Cloud into AWS.  This time we introduced ingress records in the new infrastructure that would proxy traffic back to Google Cloud. We shifted DNS to point to the new AWS ingress, then migrated customers in controlled batches, spinning up workspaces in AWS and flipping their specific ingress records one by one.

I later documented this migration pattern in a blog post: [A K8s Migration Concept](https://aaronogle.dev/2020/05/01/a-k8s-migration-concept/). The migration was completed on **March 1st, 2019**. 

### 2020: AWS to OVH

In 2020 we needed to decrease costs. So we went on the hunt to find options cheaper than AWS.  We entertained a few options from other cloud providers like Oracle offering us credits. 😅  Our search led us to **OVH** and bare metal servers which we found out even over provisioning we would be saving a lot of money and be getting the raw performance of the metal.

This brought on a lot of learning.  Here are some of the new challenges:
* No Managed Kubernetes - Rancher actually fits this particular problem very well.
* Public interface + Private Interface on a "vrack" - OVH gives you essentially a vlan on the private interface.  No gateway, no DHCP server.  So you have to manually assign an ip address to every server to be able to communicate over the private network. 
* Bastions for access
* OVH "Firewall" - So big surprise their firewall rules only protected against traffic from *outside* OVH, meaning other OVH customers could bypass them entirely. Hello iptables.
* Kubernetes Volumes - On managed cloud you just request a volume and it appears. On bare metal there's nothing. We found **Longhorn**. This let us have persistent volumes from some nodes we provisioned with a raid disk array. This mostly worked well. 

We executed **another zero-downtime migration**, completing it on **July 3rd, 2020**. This migration we refined our process further. This time exposing many ingress switching functionalities as API calls within Fleetcommand. Run ingress on both AWS and OVH. OVH pointing over to AWS for ingress. Then flipping dns and programatically moving the load and then switching the ingress to point to the OVH load.

OVH also taught us about hardware failures. Nodes just die on bare metal and way more often than we thought. CPU overheating and bootlooping randomly. Water cooling manifold related issues, thermal paste etc. Not things we expected to be having to raise support tickets and deal with. You also learn quickly about capacity planning when there's no auto-scaling button. You learn to have spares ready. You build close relationships with your vendors.

Throughout all of this... three major migrations across multiple cloud providers.. All while customer demands kept growing. We accomplished this with a small team of 3-4 people. 

In hindsight.. OVH != Traditional Bare Metal with white glove service.  I think us getting a few racks in a datacenter would have been a better match to our expectations.

## 2021: The Spotify Era

In early **2021** the company adopted the Spotify-style organizational model. A couple of Tribes.  Squads were formed with scrum procedures.  Every squad got a Product Manager, Engineering Manager, and Tech Lead. Chapters were formed for disciplines like frontend, backend and infrastructure.

Up until this point I'd been doing everything: writing code, managing infrastructure, and being the team lead. Running 1:1s, performance reviews, salary negotiations, hiring. I was listed as the hiring manager for the whole cloud team. It was a lot. 

The squad reorg changed that. What had been "the cloud team" became a cloud tribe with multiple squads. I moved into the **Architect role across the Cloud tribe**. This tribe was made up of 3 squads. The **Cloud Service Squad** (Fleetcommand, cloud portal, push gateway, and others), the **Marketplace Squad** and the **Infrastructure Squad**. I also acted as the Tech Lead for the Cloud Service Squad helping run the scrum cycles.

With all these new squads we had new engineers join like Debdut and Daniel, both coming in as interns (and still at Rocket.Chat as of today). The team was bigger than ever. **Gabriel Casals** came on as an Engineering Manager for the Cloud squads. This was a turning point for me. He was the first non-technical EM I worked closely with. He helped show me what the role should actually look like. It was like a partnership.  I helped set the technical direction and he focused on the people side. That partnership taught me how a strong EM and a strong tech lead can complement each other instead of stepping on each other's toes. 

## 2022 - 2023: Floating, Splitting, and start of the SRE Transformation

**2022** brought major changes. Our PM for the Cloud tribe left in January. **Julio Bertelli** joined. The company went through a restructuring that impacted 12 people.

I found myself increasingly acting as a **floating architect** across squads rather than being embedded in one. 

The Marketplace Squad was dissolved and absorbed into the Cloud Services Squad. It was then renamed to the **Connectivity Squad** under Gabriel Casals as EM, handling registration, licensing, marketplace, and workspace connection services. The Infra team we started calling the **SRE team**.

The catalyst for the SRE transformation was the launch of **Premium and Dedicated offerings on February 2nd, 2023**, with a commitment to a **99.9% SLA**. That kind of promise changes how you think about everything. It's not "best effort" anymore. You need monitoring, runbooks, incident response, post-mortems, on-call rotations. You need to think like an SRE team, not just an infrastructure team.

This was probably the most intense period of my career. I was back to being the architect + technical lead of both squads. Setting direction for the SRE team on the infrastructure side and the Connectivity Squad on the services side. Each with their own EM handling the people management. I became the common thread between the two.  Bridging two squads that interacted a lot.  The person who understood how the infrastructure and the services fit together end-to-end. We were reshaping what had been a purely infrastructure-focused group into a team that thought about reliability as its core mandate.

### Writing Things Down: RFCs, RFDs, ADRs, and Post-Mortems

One of the things I'm most proud of across my entire time at Rocket.Chat isn't code; it's process. Specifically, the evolution of how we made and documented technical decisions.

It started in **November 2018** when I created **[RFC 1] Self Hosted Registering with Rocket.Chat Cloud** on our internal Discourse forum. The idea was borrowed from the open-source world: before you build something significant, write it down. Explain the problem, propose solutions, invite discussion.

That first RFC kicked off a real discussion, 13 posts from me, Gabriel Engel, Guilherme Gazzo, Rodrigo, and others. RFC 2 (Cloud Multi-Region) followed two days later. Then RFC 3 (Marketplace Licensing) in February 2019. Over the next few years, 39 RFCs were written on Discourse covering everything from marketplace subscriptions to the pros and cons of moving away from Meteor to airgapped workspace registration. I personally authored about a dozen of them, mostly around cloud architecture, marketplace, and workspace connectivity.

The post-mortem practice started around the same time. My first one was the **Cloud Workspace Outage of May 2nd, 2018**. Then the Community Server Outage. Then the Push Gateway Outage. We wrote them up on the same Discourse forum, creating an internal record of what went wrong and what we learned. The post-mortem practice started on our Discourse forum with around 23 incident write-ups between 2018 and 2022. When we moved to GitHub, we imported those and kept going, ending up with roughly 45 unique post-mortems across both. Cloud disruptions, certificate expirations, database elections, a Game Day DR exercise where we discovered our backup credentials were broken.

By **2021**, the RFC process had evolved. We started calling them **RFDs** (Requests for Discussion), a subtle but meaningful shift. RFCs felt like they were asking for permission. RFDs were asking for engagement. I migrated everything from Discourse into a GitHub repo, 18 documents in a single day, pulling together the existing RFCs about microservices architecture, moving away from Meteor, the identity service, and more into one place where they could live alongside the code.

But having them in GitHub wasn't enough. ADRs were still scattered across different team folders in GitBook. Someone would share a link to an ADR and you'd never find it again. The problem was discoverability.

So in **August 2023** I built [rfd-tool](https://github.com/geekgonecrazy/rfd-tool), a small Go service that read from the GitHub repo and published to **adr.rocket.chat**. A searchable, tagged, browsable dashboard of every architectural decision. You could filter by project, by author, by tag. In that single month we pushed through 13 ADRs: License v3, RabbitMQ to NATS, Statistics Enforcement, Cloud Credential Storage, and more. The naming settled on **ADR** (Architecture Decision Record) as the umbrella term, with the understanding that discussion was always implied.

The tool kept evolving right up to the end. I added support for D2 diagrams, switched the backend from BoltDB to SQLite, added the ability to create ADRs from the web UI, and eventually added support for making specific ADRs public. One of my last PRs at the company was adding GitHub PR links to each ADR to make collaboration easier.

RFC → RFD → ADR. Discourse → GitHub → adr.rocket.chat. The format changed, the tools changed, but the core idea stayed the same from RFC 1 in 2018 to ADR 148 in 2025: write it down, invite discussion, make it discoverable.

## 2024 - 2025: Stepping Away and Coming Back

In mid-**2024**, after nearly nine years, I decided to step away. I transitioned out in July 2024 and stayed on in an advisory capacity, keeping a connection to the team and the platform I'd helped build.

The time away was valuable. I found balance. I picked up hobbies outside of work, something I hadn't really had in years. But I also realized how much I missed the work itself. Building things that matter. Solving hard problems with good people. I [wrote more about that period](https://aaronogle.dev/2024/12/31/year-in-review-2024/#work--professional) in my 2024 year in review.

On **May 19th, 2025**, I officially returned as **Head of Infrastructure & Deployment**.

This time I took both People and Tech hats deliberately. Before joining I asked exectly what the expectations were for me in the role.  I then proposed a Role Charter. Something I plan to repeat going forward. No split between a technical lead and an engineering manager. I'd seen what happened when those roles were separated and the partnership didn't gel. Simpler is better.

## 2025 - 2026:The SRE Team: Building Something That Would Outlast Me

I liked the role charter so much that after I came back, one of the first things I did was write a **team charter**, a document that made clear the team's purpose, scope, and strategic objectives. We started an SRE book club. We formalized documentation practices in Confluence. I wrote a lot... Posted priority alignments every week making sure that any adjustments in priority were clearly communicated to the team, every month setting the months priorities.  The main goal was to increase alignment so it was very clear what we were working towards.  I pushed for post-mortems after every incident, and tried to build a culture where reliability wasn't just my obsession but the team's identity.

We launched **Launchpad** (our opinionated deployment tool), drove SLO definitions across services, and pushed forward on everything from Helm chart improvements to airgapped deployment support.

## By The Numbers

Looking back across the data (because of course I had AI build a tool to analyze all of it):

| | |
|---|---|
| **Years** | ~10.5 (Aug 2015 - Mar 2026) |
| **GitHub PRs** | 1,520 across 84 repositories |
| **PR Review Comments** | 2,885 (lower then I expected.  But we dog fooded a bit heavy and tilted towards feedback often times inside of Rocket.Chat |
| **Rocket.Chat Messages** | 359,980 |
| **Rooms Active In** | 1,665 |
| **Confluence Pages Created** | 98 (Note we migrated across 3 different documentation tools over the years so this number is just for confluence since it was adopted in 2024) |
| **Confluence Edits** | 221 |
| **Zero-Downtime Migrations** | 4 (Rackspace→GCP, GCP→AWS, AWS→OVH, OVH→AWS) |
| **Cloud Providers Operated On** | 5 (Rackspace, Google Cloud, AWS, Oracle, OVH) |
| **Post-Mortems** | ~45 unique (This isn't how many I wrote but total over the years.  We started this process in 2018) |
| **RFCs/RFDs/ADRs** | 148 total, ~30 authored by me |
| **Services & Tools Built/Contributed To** | Fleetcommand, cloud-portal, Push Gateway, Marketplace API, marketplace-worker, marketplace-ui, statuscentral, statscollector, omni-gateway, launchcontrol, airlock, Launchpad, billing-service, sponsorship-service, rfd-tool, filestore-migrator, portmaster, Go SDK, FederationHub, and many more |
| **Team Names** | Cloud → Cloud Service Squad → SaaS Squad → Connectivity Services → SRE |

My peak year in the data was **2020**: 199 PRs and 29,097 Rocket.Chat messages.  I'd contribute a large part of the total to OVH migration. The year I was building, reviewing, deploying, and firefighting simultaneously. It was a massive learning curve for us.  A small team of 3-4 people running an entire cloud platform.

The relationship that shows up most clearly in the data is with **Rodrigo Nascimento**, the CTO. 8,324 DMs over the years, peaking at 2,407 messages in 2020. That was the builder-manager relationship during the most intensive infrastructure work. He believed in me when I was just a community contributor, and that belief shaped my entire career.

## What I've Learned

**Start by showing up.** I got here by fixing a bug. Then another. Then another. Nobody gave me a role. I just kept contributing until the role found me.

**Small teams can move mountains.** Three to four people executed four zero-downtime migrations across five cloud providers. We didn't wait for permission or headcount. We just figured it out.

**Titles don't matter as much as you think.** I was a "Cloud Architect" with no official title. An "Engineering Manager" who didn't know his own job description. A "Staff Engineer" who also managed people. Through all of it, the work was the same: understand the system, set direction, build the thing, help the people.

**Take the break when you need it.** I left burned out and came back with perspective. The hobbies I found during that gap made me a better leader when I returned.

**Build things that outlast you.** The team charter, the documentation culture, the post-mortem practice, the SRE book club. Those matter more than any code I wrote. Code gets replaced. Culture persists.

## Thank You

To everyone I've worked with over these 10 years, thank you. There are too many people to name, but a few I want to call out:

Bradley, for jumping into this crazy startup with me. Having a friend along for the ride made it a lot more comfortable to take that leap, and together we kept the cloud alive in those early days. Lucia, for keeping the infrastructure running when it was just us. Joseph, my favorite designer and an absolute joy to collaborate with during the squad era. Daniel, Debdut, and Paul on the SRE team. I'm proud of what we built together.

Gabriel Casals, for showing me what a great EM looks like. You set the example for how to genuinely care for the people on your team. Rodrigo, Gabriel Engel, and Chris Skelly, for the trust at every stage.

Guilherme Gazzo, for the partnership across engineering. Sing Li, for being a constant sounding board from the very beginning, always encouraging me to write more and take the leaps into the unknown that the startup world demands. Julio Bertelli "the Professional," one of the most talented Go developers I've had the chance to work with. Douglas Gubert, for the many Ketchup sessions where we'd talk about anything and everything. And Julio Araujo, the coolest head of security I've ever worked with, and one of the most talented security engineers too.

A big part of what makes Rocket.Chat special comes from the founders themselves. There's a warmth that Gabriel Engel and Rodrigo, Diego, Marcelo have always brought to this project from day one.  Always welcoming!  I remember the first time meeting everyone in person and receiving big hugs like we'd known each other for years. That warmth is something you can feel throughout the entire company. It's invisible on paper but you feel it in every interaction, and honestly it's what kept me going through all the hard times.

It's been an incredible ride. From community contribution to Head of Infrastructure & Deployment. From 1 repo to 84. From fixing UI bugs to architectural decisions that shaped how tens of thousands of workspaces run.

Time for the next adventure!

*- Aaron Ogle*
