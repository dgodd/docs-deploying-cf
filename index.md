---
title: Overview of Deploying Cloud Foundry
owner: Release Integration
---

Cloud Foundry is designed to be configured, deployed, managed, scaled, and
upgraded on any cloud IaaS provider. Cloud Foundry achieves this by leveraging
[BOSH](https://bosh.io/), an open source tool for release engineering,
deployment, lifecycle management, and distributed systems monitoring.

At a high level, the steps are the same regardless of IaaS:

1. Set up all external dependencies, such as IaaS account, external load balancers, DNS records, and any 
additional components.
2. Create a manifest to deploy a BOSH Director.
3. Deploy the BOSH Director.
4. Create a manifest to deploy Cloud Foundry.
5. Deploy Cloud Foundry.

We also offer a quick `vagrant up` solution for steps 1-3 to let you focus on pushing apps or hacking on 
Cloud Foundry itself.

Select one of the core supported infrastructures below to get started:

* <a class="subnav" href="./aws/index.html">AWS</a>
* <a class="subnav" href="./openstack/index.html">OpenStack</a>
* <a class="subnav" href="./vsphere/index.html">vSphere</a>
* <a class="subnav" href="./vcloud/index.html">vCloud Air or vCloud Director</a>

Or, if you just want the `vagrant up` experience, use BOSH-Lite:

* <a class="subnav" href="./boshlite/index.html">BOSH-Lite</a>
