---
title: Preparing to Deploy on OpenStack
owner: Release Integration
menu:
  main:
    Name: Preparing to Deploy on OpenStack
    identifier: deploying-cf/openstack
    parent: deploying-cf
---



## Step 1: Environment Setup ##

* [Validate your OpenStack Instance](validate_openstack.html)
* [Security Group for Cloud Foundry on OpenStack](security_group.html): Create a Security Group
* [DNS Setup for Cloud Foundry on OpenStack](set-up-dns.html)
* [Required Instance Flavors for Cloud Foundry on OpenStack](required-flavors.html): Confirm that your OpenStack instance has the required instance flavors.

## Step 2: Deploy BOSH ##

* [Initializing BOSH environment on OpenStack](https://bosh.io/docs/init-openstack.html).

## Step 3: Deploy Cloud Foundry ##

* [Customizing the Cloud Foundry Deployment Manifest for OpenStack](cf-stub.html)
* [Deploying Cloud Foundry using BOSH](../common/deploy.html): Deploy!

## Advanced and Support Topics ##

* [Troubleshooting Cloud Foundry on OpenStack](troubleshooting.html)
* [Using OpenStack Swift as a Cloud Foundry Blobstore](using_swift_blobstore.html)