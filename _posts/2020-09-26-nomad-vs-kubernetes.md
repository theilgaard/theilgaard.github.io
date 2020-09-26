---
layout: post
title: "Container Orchestration: Nomad or Kubernetes..."
date: 2020-09-26 +0100
categories: devops
---
I recently had the opportunity to dive down into [Hashicorp's Nomad](https://www.hashicorp.com/products/nomad) and [Kubernetes](https://kubernetes.io/). We are looking into replacing a manually provisioned setup that is currently configured by [Salt](https://www.saltstack.com/). While Salt already offers us easy deployment and configuration of all our environments from a Salt server, we lack rolling updates, horizontal scaling and an easy way to roll back to an earlier version. We also want to do cool stuff, like deploy all of our features in pull requests for our non-developer teams to review. 

Last but not least, being a small team, we would also like to avoid becoming system administrators of a small cluster of linux servers, requiring hardening and maintenance. This made us have a keen eye towards managed services, and becoming truly [Cloud Native™](https://en.wikipedia.org/wiki/Cloud_native_computing).

Our application is already containerized with docker, so picking a container orchestrator to handle deployment is the default move, it is just a matter of which one. 

# Container Orchestrators

## Why not Docker Compose or Docker Swarm?
We have built the entire application stack using docker containers, both provided by images readily available on dockerhub and our own containerized code. This led us to a natural usage of docker-compose to orchestrate our stack when developing. The entire application stack includes rabbitmq, redis, celery and our product based on [django](todo:link) fronted by a gunicorn wsgi. 

Docker Compose is great as it allows us to spin up all of our containers and dependent applications in unison. It is simple to configure and we keep our compose file source controlled in the same repository as our code. However, while it is [certainly possible to use compose in production](https://docs.docker.com/compose/production/), we were hesitant as it doesn't quite qualify as an orchestrator, lacking replication, load balancing or rolling updates. 

The next natural thing to explore would be [Docker Swarm](todo:link). Swarm is native to docker and fulfills almost all of our requirements, aside from being managed by a provider. We did however steer clear off it after discussing potential requirements not involving docker containers.

# Nomad
My team, and especially the CTO, is not afraid of adopting new tech and likes to emphasize the fun-factor of working with software. So while Nomad currently is nowhere to be seen as a service on any of the big cloud providers, it deserved a thorough investigation before being cast aside. Especially considering Hashicorp's track record of delightful products like [Terraform](https://www.hashicorp.com/products/terraform) and [Vagrant](https://www.vagrantup.com/). 

## Getting to know Nomad
Nomad offers great terminology, and when learning new software, terminology and to some degree analogy is key. Nomad uses Job, Task Groups and Tasks when defining what should actually be run on the cluster. This is great, as it immediately conveys an inheritance relationship and instills some sort of workflow in the mind of the user.

Example Nomad job file:

{% highlight json %}
job "example" {
  datacenters = ["dc1"]
  type = "service"
  group "caching-group" {
    task "redis" {
      driver = "docker"
      config {
        image = "redis:3.2"
      }
    }
    task "rabbitmq" {
      driver = "docker"
      config {
        image = "rabbitmq:latest"
      }
    }
  }
}
{% endhighlight %}

After reading some documentation, writing a simple job file, deploying it to your local development cluster, you open up the dashboard. Here, Nomad is a clear winner again. It offers an intuitive and inviting user interface, that quickly explains to you what you have running on your cluster and it's current state.

Writing a job specification does however feel unpolished, as I was left with having to [duplicate environment variables between Tasks](https://github.com/hashicorp/nomad/issues/2811) if multiple containers needed the same env vars.

Nomad's container networking is also in need of some love. As with Kubernetes, Nomad offers an extra layer of networking to isolate and communicate between task allocations. However, as Container Network Interfacing just [made it in this July](https://www.hashicorp.com/blog/announcing-general-availability-of-hashicorp-nomad-0-12), it unfortunately ended up scaring me off as being too bleeding edge.


# Kubernetes
Kubernetes is ubiquitous. It is everywhere, on all the cloud providers as a service. It is hugely popular and with that popularity comes add-ons, modifications and a ton of guides and experiences to lean on. It does however also come with a legacy.

## Getting to know Kubernetes
Deployments, Jobs, ReplicaSets, ReplicationController, Pods, etc. It just doesn't explain itself as nicely as terminology in Nomad. It requires the user to painstainkingly read all of the documentation before getting an idea about how they are connected, or not. This is of course only an issue when first learning Kubernetes.

Scowering through the ['Getting Started'](https://kubernetes.io/docs/setup/) and ['Concepts'](https://kubernetes.io/docs/concepts/) documentation of Kubernetes however, it also becomes apparent that it is wildly unopionated. This further slows my "time-to-market", and I am left rather overwhelmed.

Now where Kubernetes shines: it is wildly unopinionated! You can configure and connect your resources (or kubernetes objects depending on the page of documentation you are on) in a multitude of ways. You can control your cluster in a multitude of ways, and you can write your yaml split into multiple files or in one big file if that fits your fancy.

Kubernetes is huge. I think it is described best by the quote on [Ubuntu's website](https://ubuntu.com/kubernetes/what-is-kubernetes):
```
Kubernetes ... started as a simple container orchestration tool but has grown into a cloud-native platform. 
```

The deep cloud integration also means that spinning up a [Service](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) of type LoadBalancer actually creates a Load Balancer instance on your chosen cloud provider. How wild is that? 

In 2020, it is [unfair to call Kubernetes a Container Orchestrator](https://twitter.com/ktheilgaard/status/1304703249610571777), it is simply so much more. It is a standardization of interacting with a cloud provider, and thus must be unopionated to allow for some inversion of control.

# Conclusion
Nomad is far from mature and is simply not ready for a customer-facing production workload in my humble opinion. It already has large early adopters, but it appears to be for internal use. Had the networking and task configuration felt complete, as well as perhaps being offered as a managed service, I would probably have preferred working with Nomad, simply due to it being more opionated and thus simpler.

It is great to see the [rapid development](https://github.com/hashicorp/nomad) that Nomad is undergoing, and I look forward to seeing it as a managed service sometime in the future.

For now though, Kubernetes is king.