---
title: DNS Setup for Cloud Foundry on OpenStack
owner: Release Integration
menu:
  main:
    Name: DNS Setup for Cloud Foundry on OpenStack
    identifier: deploying-cf/openstack/set-up-dns
    parent: deploying-cf/openstack
---



Provision an IP address and set your DNS to map `*` records to this IP
address. For example, if you use `mycloud.com` domain as the base domain for
your Cloud Foundry deployment, set a `*` A record for this zone mapping to
the IP address that you provision.
