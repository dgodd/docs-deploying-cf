---
title: Validate your OpenStack Instance
owner: Release Integration
menu:
  main:
    Name: Validate your OpenStack Instance
    identifier: deploying-cf/openstack/validate_openstack
    parent: deploying-cf/openstack
---



This topic helps you validate your target OpenStack in preparation for installing BOSH and deploying Cloud Foundry.

You need a running OpenStack environment. 

<p class="note"><strong>Note</strong>: Cloud Foundry supports OpenStack releases newer than <a href="https://wiki.openstack.org/wiki/ReleaseNotes/Folsom">Folsom</a>.</p>

You can validate your OpenStack instance automatically or manually. To validate automatically, use the [cf-openstack-validator](https://github.com/cloudfoundry-incubator/cf-openstack-validator), maintained by the BOSH OpenStack CPI team. To validate manually, follow the procedures in this topic.

The manual validation procedures are considered necessary but not sufficient for BOSH to be able to use your OpenStack deployment. If you cannot perform any one of these tasks successfully, BOSH will not work; however, satisfying all these requirements does not ensure that BOSH will work.

See [Troubleshooting Cloud Foundry on OpenStack](./troubleshooting.html) for
additional troubleshooting information.

## <a id="api_access"></a>Can you access the OpenStack APIs for your instance of OpenStack? ##

You should verify that you have your OpenStack API credentials and can make API calls. Credentials are a combination of your user name, password, and the tenant or project your cloud is running under. Some providers also require you to set the region.

Create a `~/.fog` file and copy the below content:

<pre class="yaml">
:openstack:
  :openstack_auth_url:  http://HOST_IP:5000/v2.0/tokens
  :openstack_api_key:   PASSWORD
  :openstack_username:  USERNAME
  :openstack_tenant:    PROJECT_NAME
  :openstack_region:    REGION # Optional
</pre>

<p class="note"><strong>Note</strong>: You need to include <code>/v2.0/tokens</code> in the auth URL above.</p>

Install the `fog` application in your terminal, then run it in interactive mode:

<pre class="terminal">
$ gem install fog
$ fog openstack
>> Compute[:openstack].servers
[]
</pre>

The `[]` is an empty array in Ruby. You might see a long list of servers being displayed if your OpenStack tenancy/project already contains provisioned servers.

<p class="note"><strong>Note</strong>: It is recommended that you deploy BOSH and Cloud Foundry in a dedicated tenancy. This way, it is easier to keep track of the servers, volumes, security groups, and networks that you create. It also allows you to manage user access.</p>

For more information about OpenStack APIs, see the [OpenStack API Quick Start Guide](http://docs.openstack.org/api/quick-start/content/).


## <a id="metadata_service"></a> Can you access OpenStack metadata service from a virtual machine? ##

According to [the OpenStack Documentation](http://docs.openstack.org/grizzly/openstack-compute/admin/content/metadata-service.html), the Compute service uses a special metadata service to enable virtual machine instances to retrieve instance-specific data. The default stemcell for use with BOSH retrieves this metadata for each instance of a virtual machine that OpenStack manages in order to get some data injected by the BOSH director.

You must ensure that virtual machines you boot in your OpenStack environment can access the metadata service at http://169.254.169.254.

From your OpenStack dashboard, create a VM and open the console into it through the **Console** tab on its **Instance Detail** page. Wait for the terminal to appear and login.

Then execute the curl command to access the above URL. You should see a list of dates similar to the example below.

<pre class="terminal">
$ curl http://169.254.169.254

1.0
2007-01-19
2007-03-01
2007-08-29
2007-10-10
2007-12-15
2008-02-01
2008-09-01
2009-04-04

</pre>

If you do not see a list like this, consult the OpenStack documentation, or the documentation for your OpenStack distribution, to diagnose and resolve networking issues.

If the metadata service is not enabled in your OpenStack instance, you can also try [inserting instance specific data](http://docs.openstack.org/grizzly/openstack-compute/admin/content/instance-data.html#inserting_metadata) such as injecting a file when booting a VM:

<pre class="terminal">
$ gem install fog
$ fog openstack
>> s = Compute[:openstack].servers.create(name: 'test', flavor_ref: , image_ref: , personality: [{'path' => 'user_data.json', 'contents' => 'test' }])
SyntaxError: (irb):5: syntax error, unexpected ','
...ate(name: 'test', flavor_ref: , image_ref: , personality: [{...
...                               ^
(irb):5: syntax error, unexpected ')', expecting $end
  from /Users/rex/.gem/ruby/1.9.3/gems/fog-1.18.0/bin/fog:76:in `block in &lt;top (required)&gt;'
  from /Users/rex/.gem/ruby/1.9.3/gems/fog-1.18.0/bin/fog:76:in `catch'
  from /Users/rex/.gem/ruby/1.9.3/gems/fog-1.18.0/bin/fog:76:in `&lt;top (required)&gt;'
  from /Users/rex/.gem/ruby/1.9.3/bin/fog:23:in `load'
  from /Users/rex/.gem/ruby/1.9.3/bin/fog:23:in `&lt;main&gt;'
</pre>

## <a id="private_ping"></a> Can you ping one virtual machine from another? ##

Cloud Foundry requires that virtual machines be able to communicate with each other over the OpenStack networking stack. If networking is misconfigured for your instance of OpenStack, BOSH may provision VMs, but the deployment of Cloud Foundry will not function correctly because the VMs cannot properly orchestrate over NATS and other underlying technologies.

Try the following to ensure that you can communicate from VM to VM:

Create a security group for your virtual machines called **ping-test**.

1. Open the OpenStack dashboard, and click on **Access & Security** in the left-hand menu.
1. Click **Create Security Group** on the upper-right hand corner of the list of security groups.
1. Under **Name**, enter **ping-test**. Enter **ping-test** in the **Description** field.
1. Click **Create Security Group**.
1. The list of security groups should now contain **ping-test**. Find it in the list and click **Edit Rules**.
1. The list of rules should be blank. Click **Add Rule**.
1. For **Rule**, select **Custom ICMP Rule**.
1. For **Type**, enter **-1**.
1. For **Code**, enter **-1**.
1. For **Remote**, select **Security Group**.
1. For **Security Group**, select **ping-test (Current)**.
1. Click **Add**.

<p class="note"><strong>Note</strong>: If your interface contains the <strong>Direction</strong> field, use the default <strong>Direction</strong> entry to create an <strong>Ingress</strong> rule. You must create an <strong>Egress</strong> rule that matches the <strong>Ingress</strong> rule settings.</p>


From your OpenStack dashboard, create two VMs and open the console into one of them through the **Console** tab on its **Instance Detail** page. Make sure that you put these virtual machines into the **ping-test** security group. Wait for the terminal to appear and login.

Look at the list of instances in the OpenStack dashboard and find the IP address of the other virtual machine. At the prompt, issue the following command (assuming your instance receives the IP address <code>198.51.100.2</code>:

<pre class="terminal">
$ ping 198.51.100.2
PING 198.51.100.2 (198.51.100.2) 56(84) bytes of data.
64 bytes from 198.51.100.2: icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from 198.51.100.2: icmp_seq=2 ttl=64 time=0.048 ms
64 bytes from 198.51.100.2: icmp_seq=3 ttl=64 time=0.080 ms
...

</pre>

Note that you can press `Ctrl-C` to exit the ping program. If you are not able to ping one virtual machine from another, refer to the [OpenStack Networking Guide](http://docs.openstack.org/admin-guide-cloud/content/ch_networking.html) for more information.

## <a id="api_access"></a> Can you invoke large numbers of API calls? ##

Your OpenStack might have API throttling. API throttling can cause BOSH requests to OpenStack to fail while waiting for the API throttle to expire.

Create a VM and use the following commands to determine if you are affected by API throttling:

<pre class="terminal">
$ gem install fog
$ fog openstack
>> vm = Compute[:openstack].servers.get(VM_UUID)
>> 100.times { |i| vm.metadata.update('counter' => "#{i}") }
</pre>

If you are limited by API throttling, you will receive a 413 HTTP response.

To remove the API throttle if you are running **devstack**, add the following to your `localrc`:

<pre class="bash">
API_RATE_LIMIT=False
</pre>

## <a id="volumes"></a> Can you create and mount a large volume? ##

The [devstack](http://devstack.org/) OpenStack distributions defaults to a very small total volume size (5G). Alternately, your tenancy/project might have only been granted a small quota for volume sizes. You will also want to check that you can access a volume from a virtual machine to ensure that the OpenStack Cinder service is operating correctly.

To verify the ability to provision large volumes, perform the following steps:

1.  Login to your OpenStack dashboard.
2.  Click **Volumes** from the menu on the left.
3.  Click **Create Volume**.
4.  For **Volume Name**, enter "Test Volume".
5.  Put something in the **Description** field.
6.  It does not matter what you put in the **Type** field.
7.  For size, enter **30**.
8.  Click **Create Volume**.
9.  You should see the volume appear in the list of volumes with the status **Available**.

If the volume appears with the status **Error**, you need to check that your OpenStack Cinder Service is configured correctly. See [Cinder Administrator Guide](http://docs.openstack.org/admin-guide-cloud/content/managing-volumes.html) for more information.

To verify that you can attach and mount a volume to an instance, perform the following steps (**assumes you have completed the steps above**):

1.  From your OpenStack dashboard, create a VM.
2.  Return to the **Volumes** page, and find **Test Volume**. Click **Edit Attachments** on the right.
3.  In the **Attach to Instance** find the VM you just created.
4.  In the **Attach** field, enter **/dev/vdb**.
5.  Open the console into this virtual machine through the "Console" tab on its "Instance Detail" page.
6.  At the prompt, use the `fdisk` command to create the necessary partitions.
    <pre class="bash">
    $ sudo fdisk -l

	Disk /dev/vdb: 32.2 GB, 32212254720 bytes
	16 heads, 63 sectors/track, 62415 cylinders, total 62914560 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x00000000

	Disk /dev/vdb doesn't contain a valid partition table
	</pre>

Next, create a master partition on the disk by entering the bolded entries below when prompted:
<pre class="bash">
$ sudo fdisk /dev/vdb

Command (m for help): <strong>n</strong>
Command action
   e   extended
   p   primary partition (1-4) <strong>p</strong>
Partition number (1-4): <strong>1</strong>
First cylinder (1-62415, default 1): <strong>ENTER</strong>
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-62415, default 62415):<strong>ENTER</strong>
Command (m for help): <strong>t</strong>
Partition number (1-4): <strong>2</strong>
Hex code (type L to list codes): <strong>83</strong>
Changed system type of partition 2 to 83
Command (m for help): <strong>w</strong>
</pre>

Next, create a file system on the disk by entering at the prompt:
<pre class="bash">
$ sudo mkfs.ext3 /dev/vdb1
</pre>

Then, mount the disk to a directory on your VM.
<pre class="bash">
sudo mkdir /disk
sudo mount -t ext3 /dev/vdb1 /disk
</pre>

And check that you can write a file to the disk:
<pre  class="bash">
cd /disk
sudo touch pla
</pre>

If you are running **devstack**, add the following to your `localrc`:

<pre class="bash">
VOLUME_BACKING_FILE_SIZE=70000M
</pre>

## <a id="cloud-image"></a> Can you upload and deploy an Ubuntu 14.04 64-bit Server Cloud Image? ##

BOSH uses [stemcells](http://bosh.io/docs/terminology.html#stemcell) as the basis for virtual machine instances that it deploys to various cloud providers. For the OpenStack cloud provider, the BOSH stemcell is based on Ubuntu 14.04 64-bit Server. If you cannot upload this image to the Glance Image Service in your instance of OpenStack, the BOSH director will also have trouble when it tries to upload the stemcell. Similarly, if you cannot boot a virtual machine from this instance and connect to it via SSH, BOSH will also have trouble doing the same.  Additionally, you will want to check that the underlying hardware that runs your OpenStack is compatible with running a 64-bit operating system.

To check that your OpenStack is compatible with Ubuntu 14.04 64-bit Server, you will need to download the image to your Glance Image Service, then separately boot the image as an instance in OpenStack.

To download the image to your Glance Image Service, perform the following steps:

1.  Login to your OpenStack dashboard.
2.  In the menu on the left-hand side, click **Images & Snapshots**.
3.  You should see a list of images. Click **Create Image**.
4.  For **Name**, enter **Ubuntu Server 64-bit**.
5.  For **Image Location**, enter **http://cloud-images.ubuntu.com/lucid/current/lucid-server-cloudimg-amd64-disk1.img**.
6.  For **Format**, select **QCOW2 - QEMU Emulator**.
7.  For **Minimum Disk**, enter 5GB.
8.  For **Minimum Ram**, enter 1024MB.
9.  Click **Create Image**.

After the image has download, launch an instance of it from the dashboard. If the image seems to take a significantly long amount of time to boot, it may be that your metadata service is not configured correctly.

## <a id="internal-external-ips"></a> Can networking be configured for both external and internal IPs? ##

BOSH assumes that virtual machines will be connected to two distinct virtual networks: one that is internal and only accessible to the virtual machines themselves, and one that is external that allows access from outside the network. In the user interface, these external IP addresses are called **floating** IPs because they can be dynamically reassigned to virtual machines on the hypervisor with the push of a button.

To confirm that you can assign floating IP addresses to your virtual machines, perform the following steps:

1.  Find a virtual machine you have already provisioned, or provision one if  you do not have one available. In the menu on the left-hand side, if you click **Instances**, you should see your virtual machine listed.
2.  Ensure that the virtual machine in #1 has booted completely by checking its console window (the "Console" tab on its "Instance Detail" page).
4.  Find your virtual machine in the list of instances and click the button labeled **More** for that VM.  A list of operations should appear.
5.  Select **Associate Floating IP** from the list that comes up.
6.  You should be able to pull the drop down the list of public IP addresses from the menu named **IP Address**. Sometimes, you may need to allocate an additional floating IP address by clicking the plus (+) to the right of this menu, selecting a pool, and clicking **Allocate IP**.
7.  You should be able to leave the menu **Port to be associated** alone, but check that it points at the VM you have selected.
8.  Click **Associate**.

If the steps above do not work for you, consult the [OpenStack Networking Guide](http://docs.openstack.org/admin-guide-cloud/content/ch_networking.html) for more information.

Simply assigning a floating IP address to a virtual machine does not mean that virtual networking is properly configured. You will also need to ensure that the virtual machine can be reached on its floating IP address, so that calls can be made to and from the VM once it's running the BOSH Director or Cloud Foundry. The public-facing router will forward traffic at all ports to your virtual machine, so it is up to the networking configuration of your security group to provide the added layer of security by implementing a firewall.

Create a security group for your public-facing virtual machine called **cidr-ping-test**.

1.  Open the OpenStack dashboard, and click on **Access & Security** in the left-hand menu.
2.  Click **Create Security Group** on the upper-right hand corner on the list of security groups.
3.  Under **Name**, enter **cidr-ping-test**. The **Description** field requires text, but the text can be anything.
4.  Click **Create Security Group**.
5.  The list of security groups should now contain **cidr-ping-test**. Find it in the list and click the **Edit Rules** button.
6.  The list of rules should be blank. Click **Add Rule**.
7.  For **Protocol**, select **ICMP**.
8.  For **Type**, enter **-1**.
9.  For **Code**, enter **-1**.
10. For **Source**, select **CIDR**.
11. For **CIDR**, enter **0.0.0.0/0**.
12. Click **Add**.

You now need to add your virtual machine to the security group you just created.

1.  Go back to the **Instances** view and find your VM.
2.  Click the **More** button to the right of your VM.
3.  Click the **Edit Security Groups** button in the drop down menu that appears.
4.  Find **cidr-ping-test** in the list under **All Security Groups** and click the plus (+) to add it to the virtual machine.

Now that the **cidr-ping-test** group has been added to the VM and configured to allow ping traffic through, and the virtual machine has been associated with a floating IP address, you can ping the VM to check that the traffic is properly routed.

At the prompt, issue the following command (assuming your instance receives the IP address <code>9.9.8.7</code>:

<pre class="terminal">
$ ping 9.9.8.7
PING 9.9.8.7 (9.9.8.7) 56(84) bytes of data.
64 bytes from 9.9.8.7: icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from 9.9.8.7: icmp_seq=2 ttl=64 time=0.048 ms
64 bytes from 9.9.8.7: icmp_seq=3 ttl=64 time=0.080 ms
...

</pre>

## <a id="internet"></a> Can you access the Internet from within instances? ##

Your deployment of Cloud Foundry will need outbound access to the Internet. For example, the Ruby buildpack will run `bundle install` on users' applications to fetch RubyGems. You can verify that your OpenStack is configured correctly to allow outbound access to the Internet.

From your OpenStack dashboard, create a VM and open the console into it through the "Console" tab on its "Instance Detail" page. Wait for the terminal to appear and login.

<pre class="terminal">
$ curl google.com
...
&lt;H1>301 Moved&lt;/H1>
The document has moved
...
</pre>

If you do not see the output above, consult the OpenStack documentation (or the documentation for your OpenStack distribution) to diagnose and resolve networking issues.

If you are running **devstack**, check that you have an `eth0` and `eth1` network interface when running `ifconfig`. If you only have `eth1`, try adding the following lines to your `localrc` file before recreating your devstack:

<pre class="bash">
PUBLIC_INTERFACE=eth1
FLAT_INTERFACE=eth0
</pre>

<p class="note"><strong>Note</strong>: You cannot use VMs that only have ping-test security groups assigned because they do not allow Internet traffic.</p>

## <a id="internet"></a> Can you use F5 instead of HAProxy? ##

HAProxy is deployed by default as your external-facing load balancer. You can use F5 instead of HAProxy. However, setting up F5 requires advanced configuration. F5 setup instructions are not covered in this documentation. 
