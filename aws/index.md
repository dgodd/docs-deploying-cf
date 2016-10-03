---
title: Preparing to Deploy on Amazon Web Services
owner: Release Integration
---

<strong></strong>

The following steps document an example procedure for deploying Cloud Foundry on AWS. The process includes several steps that may be expensive and time-consuming, such as provisioning AWS Relational Database Service (RDS)
instances. For a simpler deployment, refer to the
[Minimal AWS](https://github.com/cloudfoundry/cf-release/tree/master/example_manifests) instructions.

## <a id='prereqs'></a>Prerequisites

To complete this deployment, you need:

- An [Amazon Web Services](http://aws.amazon.com) (AWS) account with Virtual Private Cloud (VPC)

- The [BOSH CLI](https://bosh.io/docs/bosh-cli.html) and [bosh-init](https://bosh.io/docs/install-bosh-init.html) installed on your machine

- [Ruby] (https://www.ruby-lang.org/en/documentation/installation/) 1.9.3 or higher installed on your machine

- Sufficiently high instance limits on your AWS account. Installing Cloud Foundry using this bootstrap tool configures a production environment with more than 20 component VM instances.

<p class="note"><strong>Note</strong>: You can deploy Cloud Foundry in smaller topologies, including a <a href="../boshlite/">local development environment</a> on Vagrant or a <a href="https://github.com/cloudfoundry/cf-release/tree/master/example_manifests">minimal AWS configuration</a>.</p>

##<a id="env-setup"></a> Step 1: Environment Setup ##

* [Setting up an AWS Environment for Cloud Foundry with BOSH AWS Bootstrap](setup_aws.html): Use the BOSH AWS Bootstrap tool `bosh aws create` to create an environment on AWS that includes a VPC and other structures that support BOSH.

## <a id="deploy-bosh"></a> Step 2: Deploy BOSH ##

* [Deploying BOSH on AWS](setup_bosh_aws.html): Create a manifest for BOSH, and pass it to `bosh-init` to deploy BOSH in your environment.

## <a id="deploy-cf"></a> Step 3: Deploy Cloud Foundry ##

* [Customizing the Cloud Foundry Deployment Manifest for AWS](cf-stub.html)
* [Deploying Cloud Foundry using BOSH](../common/deploy.html): Deploy!
