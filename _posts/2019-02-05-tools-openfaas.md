---
title: "Favorite Tools - OpenFaaS"
date: 2019-02-05T14:48:40+01:00
categories: ["favorite-tools", "tools"]
tags: ["Tools", "Favorite Tools", "OpenFaaS", "FaaS", "Open Source"]
readtime: true
---

"The mission of OpenFaaS ® is to Make Serverless Functions Simple. It can be run on any cloud through the use of portable Docker containers without the fear of getting locked-in or having to manage complex infrastructure."

Bold statement but just so true. [OpenFaaS](https://www.openfaas.com/) is easily one of my favorite tools and Open-Source projects out there. There are so many things on the plus side for OpenFaaS.

<!--more-->

First up it's [easy to deploy](https://docs.openfaas.com/deployment/). Install Docker, which you should do anyways, install the faas-cli, deploy the stack and you're good to go for your first function as a service. I've managed to solve that manually in under 5 minutes with a sub-optimal connection!

It's well documented and the community built around the project is one of the best OSS communities I'm aware of. A large part of this archievement comes through the founder of OpenFaaS himself - [Alex Ellis](https://twitter.com/alexellisuk) is extremly inspiring and tries to help the community as much as possible. Last year he even managed to get a job at @VMware to work full-time on OpenFaaS.

The biggest and most important feature of all is the versatility of OpenFaaS. You may run it on a single- or multi-node Docker Swarm (which is my preferred deployment) or Kubernetes Cluster or through providers even on [Hashicorp's Nomad](https://github.com/hashicorp/faas-nomad). All that on full blown bare-metal servers or small SBC like the RaspPi. Even more important, it is dead simple to write new functions - nearly in every "language"/tooling you can think of. I won't get into detail here, just follow the many good tutorials out there like [on the official docs](https://github.com/openfaas/workshop). And be sure to follow [Alex Ellis](https://twitter.com/alexellisuk) and [OpenFaaS](https://twitter.com/openfaas) Twitter accounts and Blogs for updates and more good posts.

I can just think of one sort of negative point here - the range/reach of OpenFaaS out there. IMO it may compete with the big players from Amazon to Microsoft in certain situations, especially as you may deploy it even there and on-prem but in reality it's just not has the adoption. There's always place for multiple solutions and we will have to see how far it gets. Until then have a look at it, try it, contribute to it and maybe even support it through an [backing at Patreon](https://www.patreon.com/alexellis).
