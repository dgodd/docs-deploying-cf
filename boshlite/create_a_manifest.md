---
title: Create a Deployment Manifest for Cloud Foundry on BOSH-Lite
owner: Release Integration
menu:
  main:
    Name: Create a Deployment Manifest for Cloud Foundry on BOSH-Lite
    identifier: deploying-cf/boshlite/create_a_manifest
    parent: deploying-cf/boshlite
---



This topic describes how to create the Cloud Foundry deployment manifest for BOSH-Lite. It assumes you have already [installed BOSH-Lite](https://github.com/cloudfoundry/bosh-lite/blob/master/README.md#install-bosh-lite) and the [BOSH CLI](https://bosh.io/docs/bosh-cli.html).

##<a id="create-stub"></a>Create a Deployment Manifest Stub (Optional for Virtualbox) ##

If you deploy BOSH-Lite to an AWS Vagrant VM, you must specify a system domain
in a manifest stub. If you are deploying to a local Vagrant VM this step is not
necessary.

By default, BOSH-Lite uses the system domain `bosh-lite.com`, which resolves
to the address `10.244.0.34`. Update the system domain property to one of the
following two values:

  - The domain that you configured to point to the public IP address of the
  BOSH-Lite box that you previously created.
  - The `xip.io` domain the corresponds to the public IP address given to your
  BOSH-Lite vagrant box: `YOUR_PUBLIC_IP.xip.io`. For more information about
  xip.io, see [xip.io](http://xip.io/).

```
    ---
    properties:
      system_domain: bosh-lite.com # Replace with your system domain
```

##<a id='target'></a>Target Your BOSH Director

Use the `bosh target` command with the address of your BOSH Director to
connect to the BOSH Director. Log in with the default user name and password,
`admin` and `admin`, or use the username and password that you set when you
installed BOSH-Lite.
<pre class="terminal">
$ bosh target <span>https</span>://bosh.my-domain.example.com
Target set to `bosh'
Your username: admin
Enter password: *****
Logged in as 'admin'
</pre>

##<a id="clone"></a>Clone the cf-release GitHub Repository ##

<pre class="terminal">
$ git clone https://github.com/cloudfoundry/cf-release.git
</pre>

##<a id="generate-manifest"></a>Generate the Manifest ##

Ensure that you have the most up-to-date version of the Cloud Foundry code and all required submodules.

1. From the `cf-release` directory that you cloned when you created the
manifest, run the update script to fetch all the submodules.

    <pre class="terminal">
    $ cd cf-release
    $ ./scripts/update
    </pre>

1. Install the Ruby version listed in the `.ruby-version` file of the cf-release repository. You can manage your Ruby versions using `rvm`, `rbenv`, or `chruby`.
1. Run `gem install bundler` to install `bundler`.
1. Install [spiff](https://github.com/cloudfoundry-incubator/spiff).
1. Use the `scripts/generate-bosh-lite-dev-manifest` command to create a
deployment manifest and set it as the current BOSH deployment.

    <pre class="terminal">
    $ cd cf-release
    $ ./scripts/generate-bosh-lite-dev-manifest
    </pre>
1. (Optional) If you have deployed your BOSH-Lite to AWS, or have otherwise created any stubs with custom configuration, you should pass them as additional arguments:
    <pre class="terminal">
    $ ./scripts/generate-bosh-lite-dev-manifest PATH-TO-STUB ...
    </pre>
