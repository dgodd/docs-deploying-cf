---
title: Troubleshooting Cloud Foundry on OpenStack
owner: Release Integration
---

<strong></strong>

This topic describes helpful tips to deploy Cloud Foundry on OpenStack.

## Self-signed SSL Certificates
View these two topics to add your self-signed certificates to the OpenStack CPI and to the virtual machines (VMs) deployed with your BOSH Director:

* The BOSH OpenStack CPI uses a [chain of trusted certificates](http://bosh.io/docs/openstack-self-signed-endpoints.html) to validate the certificate of your OpenStack endpoints with a property called <code>ca_cert</code>.
* The BOSH Director adds [chain of trusted certificates](http://bosh.io/docs/trusted-certs.html) to all VMs deployed with that Director with a property called <code>trusted_certs</code>.


## GRE Tunnels
If you are using OpenStack's GRE Tunnel networking, the default MTU of 1500 does not allow machines to properly communicate.  Warden is setup out of the box to use 1500 of MTU. Modify the MTU by changing the configuration file for your Cloud Foundry deployment.  

Change the MTU in your `dea_next` block:

```
dea_next:
  mtu: 1454
```

Use `bosh deploy` to push these changes and then reboot your Warden / DEA VM.

## NFS and Cloud Controller

<p class="note"><strong>Note</strong>: Cloud Foundry versions 231 and later use the WebDAV protocol instead of NFS.</p>

If you get the following error when deploying an app to Cloud Foundry you may have an NFS related issue with the Cloud Controller:
<pre class='terminal'>
$ The app package is invalid: failed synchronizing resource pool File exists - /var/vcap/nfs/shared
</pre>

To confirm that this is related to a broken or incomplete NFS mount, SSH into the `cloud_controller_ng` job and checking the existence of the `/var/vcap/nfs/shared` folder:

<pre class='terminal'>
$ bosh ssh cloud_controller/0 "ls -l /var/vcap/nfs/shared"
...
ls: cannot access /var/vcap/nfs/shared: Stale NFS file handle"
</pre>

Try the following options to resolve this issue:

1. (Recommended) Restart the `cloud_controller`, or `api`, jobs with the BOSH CLI `bosh restart cloud_controller`.
2. Manually recreate the NFS mount on the `cloud_controller` job server:

    <pre class='terminal'>
    $ bosh ssh cloud_controller/0
    root:~# umount /var/vcap/nfs
    root:~# mount -t nfs 0.nfs.default.cf.microbosh:/var/vcap/store /var/vcap/nfs
    </pre>

    Replace `0.nfs.default.cf.microbosh` above with the static IP or DNS host of the job instance running the `debian_nfs_server`.

