+++
author = "Matthew Macdonald"
title = "AKS - Dual Ingress Controllers - Private & Public"
date = "2023-05-06"
description = "How to create an AKS cluster that utlizes 2 ingress controllers - one for private access and one exposed publicly."
tags = [
    "kubernetes",
    "aks",
    "azure",
    "nginx"
]
+++

## A need for securing internal resources

One of the beauties of the modern cloud providers, is how easy they make it to spin up new resources and environments, often quicker than you can make a coffee. While this allows for very quick PoC's and IaC development - the defaults for these resources are often tailored to cause the least amount of developer impediment possible, at the expense of optimal security. Now this is generally desirable for your average PoC or dev environment - but proceed with caution when it comes to production.

Take the defaults when creating a new AKS cluster as an example: a one-line Azure CLI command as simple as `az aks create -g myResourceGroup -n myAksCluster`. This will spin up a 2 node cluster with a publicly accessible kube-apiserver and load balancer. A publicly exposed control plane is an attack vector that just isn't worth risking in a production setting, especially with how easy the alternatives can be. Without going out of your way - any ingresses created on this cluster will also be publicly accessible.

While we generally run kubernetes in order to expose our applications to our public user bases, there is also a lot that goes on in a cluster that doesnt concern the public and may even pose a security risk. Things like CICD tools, observability dashboards, cluster management frontends, or in-house applications often have no business sitting on a public IP address. Wouldn't it be nice if we had a way to expose those internally on a private frontend while our customer facing applications remained accessible publicly?

That is what we will tackle today with a little bit of Terraform & Helm

## Application Gateway or nginx ?

There are two different paths we can go down to achieve this and we will be covering both.
1. Application Gateway Ingress Controller
2. nginx Ingress Controller

{{< figure src="/images/public-private-ingress/ingress-architecture.png" title="Test Image" >}}

We wont go into the advantages of chosing one over the other - I'll leave that decision and further research up to you.

## Creating the cluster

We'll use Terraform to create our resources, starting with our AKS cluster.
You can find the code used in this blog post on my github at ...
