---
title: Customizing the Cloud Foundry Deployment Manifest for AWS
owner: Release Integration
menu:
  main:
    Name: Customizing the Cloud Foundry Deployment Manifest for AWS
    identifier: deploying-cf/aws/cf-stub
    parent: deploying-cf/aws
---



This topic describes how to create the Cloud Foundry deployment manifest for Amazon Web Services (AWS). Before creating a manifest, you must have already [set up an environment](setup_aws.html) for Cloud Foundry on AWS and [deployed BOSH](setup_bosh_aws.html) on AWS.

To create a Cloud Foundry manifest, you must perform the following steps:

1. Use the BOSH CLI to [retrieve your BOSH Director UUID](#bosh-uuid), which you use to customize your manifest stub.
1. Create a manifest stub in YAML format. See the [example manifest stub](#stub) for AWS below, and follow the [editing instructions](#editing) to customize it for your deployment.
1. Use a script to combine the manifest stub with other configuration files in the `cf-release` repository to [generate your deployment manifest](#generate).

<p class="note"><strong>Note</strong>: AWS defaults to using Fog with AWS credentials for the Cloud Controller blobstore. For alternative blobstore configurations, see the <a href="/deploying/common/cc-blobstore-config.html">Cloud Controller Blobstore Configuration</a> topic.</p>

##<a id='bosh-uuid'></a> Step 1: Retrieve Your BOSH Director UUID

<%= partial '../common/bosh_uuid' %>

##<a id='stub'></a> Step 2: Create Your Manifest Stub

Review the [example manifest stub](#example-stub) for AWS, and then follow the [editing instructions](#editing) to customize it for your deployment.

###<a id="example-stub"></a>Cloud Foundry Deployment Manifest Stub for AWS

<%= yield_for_code_snippet from: 'cloudfoundry/cf-release', at: 'cf-stub-aws' %>

###<a id="editing"></a>Editing Instructions 

<table border="1" class="nice">
  <tr>
    <th style="width:35%">Deployment Manifest Stub Contents</th>
    <th>Editing Instructions</th>
  </tr>
  <tr>
    <td><pre><code>
meta:
  environment: ENVIRONMENT
    </code></pre></td>
    <td>Replace <code>ENVIRONMENT</code> with an arbitrary name describing your environment, e.g. <code>aws-prod</code>.
    </td>
  </tr>
  <tr>
    <td><pre><code>
director_uuid: DIRECTOR_UUID
    </code></pre></td>
    <td>Replace <code>DIRECTOR_UUID</code> with the BOSH Director UUID. Use
    <code>bosh status --uuid</code> to view the BOSH Director UUID.
    </td>
  </tr>
  <tr>
    <td><pre><code>
networks:
- name: cf1
  subnets:
    - range: 10.10.16.0/20
      reserved:
        - 10.10.16.2 - 10.10.16.9
      static:
        - 10.10.16.10 - 10.10.16.255
      gateway: 10.10.16.1
      dns:
        - 10.10.0.2
      cloud_properties:
        security_groups:
          - cf
        subnet: (( properties.template_only.aws.subnet_ids.cf1 ))
- name: cf2
  subnets:
    - range: 10.10.80.0/20
      reserved:
        - 10.10.80.2 - 10.10.80.9
      static:
        - 10.10.80.10 - 10.10.80.255
      gateway: 10.10.80.1
      dns:
        - 10.10.0.2
      cloud_properties:
        security_groups:
          - cf
        subnet: (( properties.template_only.aws.subnet_ids.cf2 ))
      </code></pre></td>
    <td>If you used <code>bosh aws create</code> to set up your AWS environment, change the IP addresses here if needed to fit the subnet range from your bosh.yml file.
    <br/><br/>
    This example assumes you have two subnets in your AWS VPC with CIDRs <code>10.10.16.0/20</code> and <code>10.10.80.0/20</code> respectively. Update the values for <code>range</code>, <code>reserved</code>, <code>static</code>, and <code>gateway</code> accordingly if the CIDRs for your subnets are different.
    <br/><br/>
    This also assumes that you have a security group <code>cf</code> suitable for your Cloud Foundry VMs. Change this too if the name of your security group is different.
      </td>
  </tr>
  <tr>
    <td><pre><code>
properties:
  template_only:
    aws:
      access_key_id: AWS_ACCESS_KEY
      secret_access_key: AWS_SECRET_ACCESS_KEY
      availability_zone: ZONE_1
      availability_zone2: ZONE_2
      subnet_ids:
        cf1: SUBNET_ID_1
        cf2: SUBNET_ID_2
      </code></pre></td>
    <td>Replace <code>AWS_ACCESS_KEY</code> and <code>AWS_SECRET_ACCESS_KEY</code> with AWS credentials to allow Cloud Controller to manage assets in the S3 buckets you have prepared for this deployment.
    <br/><br/>
    Replace <code>ZONE_1</code> and <code>ZONE_2</code> with two EC2 Availability Zones that you want to distribute your deployment across.
    <br/><br/>
    Replace <code>SUBNET_ID_1</code> and <code>SUBNET_ID_2</code> with the VPC subnet IDs corresponding to the subnets configured in the <code>networks</code> section above.
    <br/><br/>
    If you used <code>bosh aws create</code>, the key and zone values are in  the <code>bosh_environment</code> file that you originally bootstrapped from and the subnet IDs correspond to the subnets cf1 and cf2 already created.
    </td>
  </tr>
  <tr>
    <td><pre><code>
  system_domain: SYSTEM_DOMAIN
  system_domain_organization: SYSTEM_DOMAIN_ORGANIZATION
  app_domains:
   - APP_DOMAIN
      </code></pre></td>
    <td>Replace <code>SYSTEM_DOMAIN</code> with the domain to be used for all system components. For instance, the Cloud Controller API will be reachable via <code>api.SYSTEM_DOMAIN</code>. Replace <code>APP_DOMAIN</code> with the domain you want associated with applications pushed to your Cloud Foundry installation. For instance, if you push <code>my-app</code> it will be reachable via <code>my-app.APP_DOMAIN</code>.
    <br/><br/>
    It is generally advised to have separate values for <code>SYSTEM_DOMAIN</code> and <code>APP_DOMAIN</code>, e.g. <code>sys.cloud-09.cf-app.com</code> and <code>apps.cloud-09.cf-app.com</code>. For simple deployments, you can use the same domain for both, e.g. <code>cloud-09.cf-app.com</code>. However, you cannot have one domain property extend the other, e.g. you cannot have your <code>SYSTEM_DOMAIN</code> set to <code>cloud-09.cf-app.com</code> and your <code>APP_DOMAIN</code> set to <code>apps.cloud-09.cf-app.com</code>.
    <br/><br/>
    If you used <code>bosh aws create</code>, use the same value for both, created by concatenating the <code>BOSH_VPC_SUBDOMAIN</code> and <code>BOSH_VPC_DOMAIN</code> values in the <code>bosh_environment</code> file.
    <br/><br/>
    Pick any name you like for the <code>SYSTEM_DOMAIN_ORGANIZATION</code>. This organization will be created and configured to own the <code>SYSTEM_DOMAIN</code>.
    </td>
  </tr>
  <tr>
    <td><pre><code>
  ssl:
    skip_cert_verify: true
    </code></pre></td>
    <td> Set <code>skip_cert_verify</code> to <code>true</code> to skip SSL certificate verification. If you want to use SSL certificates to secure traffic into your deployment, see the <a href="../../adminguide/securing-traffic.html">Securing Traffic into Cloud Foundry</a> topic.
    </td>
  <tr>
    <td><pre><code>
  cc:
    staging_upload_user: STAGING_UPLOAD_USER
    staging_upload_password: STAGING_UPLOAD_PASSWORD
    bulk_api_password: BULK_API_PASSWORD
    db_encryption_key: CCDB_ENCRYPTION_KEY
    </code></pre></td>
    <td>
      The Cloud Controller API endpoint requires basic authentication. Replace <code>STAGING_UPLOAD_USER</code> and <code>STAGING_UPLOAD_PASSWORD</code> with a username and password of your choosing.
      <br /><br />
      Replace <code>BULK_API_PASSWORD</code> with a password of your choosing. Health Manager uses this password to access the Cloud Controller bulk API.
      <br /><br />
      Replace <code>CCDB_ENCRYPTION_KEY</code> with a secure key that you generate to encrypt sensitive values in the Cloud Controller database. 
      You can use any random string. For example, run <code>md5 -qs "$(date)"</code> from a command line to generate a random string using the MD5 Hash Generator.
  </td>
  </tr>
  <tr>
    <td><pre><code>
  ccdb:
    db_scheme: CCDB_SCHEME
    roles:
      - tag: admin
        name: CCDB_USER_NAME
        password: CCDB_PASSWORD
    databases:
      - tag: cc
        name: ccdb
    address: CCDB_ADDRESS
    port: CCDB_PORT
    </code></pre></td>
    <td>
      This section of the stub defines how the Cloud Controller connects to its database. The values depend on how you deployed your database.
      <br /><br />
      If you used <code>bosh aws create</code>, find the necessary values in the generated file <code>aws_rds_bosh_receipt.yml</code>. The database is an Amazon RDS instance, and the <code>CCDB_*</code> values in the stub must match the scheme, username, password, address, and port defined by AWS.
      <br /><br />
      If you deployed your database without <code>bosh aws create</code>, such as using the <code>postgres</code> job in cf-release, the <code>CCDB_*</code> values must match the configuration of the database node. If you are using PostgreSQL, your database must have the required extensions available for Cloud Foundry: <code>uuid-ossp</code>, <code>pgcrypto</code>, and <code>citext</code>. The <code>db_scheme</code> for a PostgreSQL database is <code>postgresql</code>, not <code>postgres</code>.
      </td>
  </tr>
  <tr>
    <td><pre><code>
  consul:
    encrypt_keys:
      - CONSUL_ENCRYPT_KEY
    ca_cert: CONSUL_CA_CERT
    server_cert: CONSUL_SERVER_CERT
    server_key: CONSUL_SERVER_KEY
    agent_cert: CONSUL_AGENT_CERT
    agent_key: CONSUL_AGENT_KEY
      </code></pre></td>
    <td>See the instructions on <a href="../common/consul-security.html">security configuration for consul</a>.
      </td>
  </tr>
  <tr>
    <td><pre><code>
  loggregator_endpoint:
    shared_secret: LOGGREGATOR_ENDPOINT_SHARED_SECRET
    </code></pre></td>
    <td>Generate a string secret and replace <code>LOGGREGATOR_ENDPOINT_SHARED_SECRET</code>.
      </td>
  </tr>
  <tr>
    <td><pre><code>
  nats:
    user: NATS_USER
    password: NATS_PASSWORD
      </code></pre></td>
    <td>Replace <code>NATS_USER</code> and
        <code>NATS_PASSWORD</code> with a username and secure password of your choosing. Cloud Foundry components use these credentials to communicate with each other over the NATS message bus.
      </td>
  </tr>
  <tr>
    <td><pre><code>
  router:
    status:
      user: ROUTER_USER
      password: ROUTER_PASSWORD
      </code></pre></td>
    <td>
      Replace <code>ROUTER_USER</code> and
        <code>ROUTER_PASSWORD</code> with a username and secure password of your choosing.
      </td>
  </tr>
  <tr>
    <td><pre><code>
  uaa:
    admin:
      client_secret: ADMIN_SECRET
    cc:
      client_secret: CC_CLIENT_SECRET
    clients:
      cc-service-dashboards:
        secret: CC_SERVICE_DASHBOARDS_SECRET
      cc_routing:
        secret: CC_ROUTING_SECRET
      cloud_controller_username_lookup:
        secret: CLOUD_CONTROLLER_USERNAME_LOOKUP_SECRET
      doppler:
        secret: DOPPLER_SECRET
      gorouter:
        secret: GOROUTER_SECRET
      tcp_emitter:
        secret: TCP-EMITTER-SECRET
      tcp_router:
        secret: TCP-ROUTER-SECRET
      login:
        secret: LOGIN_CLIENT_SECRET
      notifications:
        secret: NOTIFICATIONS_CLIENT_SECRET
    </code></pre></td>
    <td>Replace all the <code>*_SECRET</code>s with secure secrets that you generate.
      </td>
  </tr>
  <tr>
    <td><pre><code>
    jwt:
      policy:
        active_key_id: key-1
        keys:
          key-1:
            signingKey: |
              -----BEGIN RSA PRIVATE KEY-----
              MIICXgIBAAKBgQDfTLadf6QgJeS2XXImEHMsa+1O7MmIt44xaL77N2K+J/JGpfV3
              AnkyB06wFZ02sBLB7hko42LIsVEOyTuUBird/3vlyHFKytG7UEt60Fl88SbAEfsU
              JN1i1aSUlunPS/NCz+BKwwKFP9Ss3rNImE9Uc2LMvGy153LHFVW2zrjhTwIDAQAB
              AoGBAJDh21LRcJITRBQ3CUs9PR1DYZPl+tUkE7RnPBMPWpf6ny3LnDp9dllJeHqz
              a3ACSgleDSEEeCGzOt6XHnrqjYCKa42Z+Opnjx/OOpjyX1NAaswRtnb039jwv4gb
              RlwT49Y17UAQpISOo7JFadCBoMG0ix8xr4ScY+zCSoG5v0BhAkEA8llNsiWBJF5r
              LWQ6uimfdU2y1IPlkcGAvjekYDkdkHiRie725Dn4qRiXyABeaqNm2bpnD620Okwr
              sf7LY+BMdwJBAOvgt/ZGwJrMOe/cHhbujtjBK/1CumJ4n2r5V1zPBFfLNXiKnpJ6
              J/sRwmjgg4u3Anu1ENF3YsxYabflBnvOP+kCQCQ8VBCp6OhOMcpErT8+j/gTGQUL
              f5zOiPhoC2zTvWbnkCNGlqXDQTnPUop1+6gILI2rgFNozoTU9MeVaEXTuLsCQQDC
              AGuNpReYucwVGYet+LuITyjs/krp3qfPhhByhtndk4cBA5H0i4ACodKyC6Zl7Tmf
              oYaZoYWi6DzbQQUaIsKxAkEA2rXQjQFsfnSm+w/9067ChWg46p4lq5Na2NpcpFgH
              waZKhM1W0oB8MX78M+0fG3xGUtywTx0D4N7pr1Tk2GTgNw==
              -----END RSA PRIVATE KEY-----
    </code></pre></td>
    <td>Generate a PEM-encoded RSA key pair. This creates <code>jwt-key.pem.pub</code>, which contains your public key, and <code>jwt-key.pem</code>, which contains your private key.<br/>
    You can generate a key pair by using the command <code>openssl rsa -in jwt-key.pem -pubout > key.pub</code>. This process creates <code>jwt-key.pem.pub</code>, which contains your public key, and <code>jwt-key.pem</code>, which contains your private key. Paste in the full private key, including the <code>BEGIN</code> and <code>END</code> delimiter lines, under <code>signingKey</code>.</br>
    For RSA keys, all that needs to be configured is the private key. You can configure as many keys as you wish. However, only one, <code>active_key_id</code>, will be used to sign tokens with.
    The others are used for signature validation.
    </td>
  </tr>
  <tr>
    <td><pre><code>
    scim:
      users:
      - name: admin
        password: ADMIN_PASSWORD
        groups:
          - scim.write
          - scim.read
          - o...
    </code></pre></td>
    <td>Generate a secure password and replace <code>ADMIN_PASSWORD</code> with that value to set the password for the Admin user of your Cloud Foundry installation.
    </td>
  </tr>
  <tr>
    <td><pre><code>
  uaadb:
    db_scheme: UAADB_SCHEME
    roles:
      - tag: admin
        name: UAADB_USER_NAME
        password: UAADB_USER_PASSWORD
    databases:
      - tag: uaa
        name: uaadb
    address: UAADB_ADDRESS
    port: UAADB_PORT
      </code></pre></td>
    <td>
      This section of the stub defines how the UAA connects to its database. The values depend on how you deployed your database.
      <br /><br />
      If you used <code>bosh aws create</code>, find the necessary values in the generated file <code>aws_rds_bosh_receipt.yml</code>. The database is an Amazon RDS instance, and the <code>UAADB_*</code> values in the stub must match the scheme, username, password, address, and port defined by AWS.
      <br /><br />
      If you deployed your database without <code>bosh aws create</code>, such as using the <code>postgres</code> job in cf-release, the <code>UAADB_*</code> values must match the configuration of the database node. If you are using PostgreSQL, your database must have the required extensions available for Cloud Foundry: <code>uuid-ossp</code>, <code>pgcrypto</code>, and <code>citext</code>. The <code>db_scheme</code> for a PostgreSQL database is <code>postgresql</code>, not <code>postgres</code>.
      </td>
  </tr>
  <tr>
    <td><pre><code>
  hm9000:
    server_key: HM9000_SERVER_KEY
    server_cert: HM9000_SERVER_CERT
    client_key: HM9000_CLIENT_KEY
    client_cert: HM9000_CLIENT_CERT
    ca_cert: HM9000_CA_CERT
    </code></pre></td>
    <td>
      Generate SSL certs for HM9000 and replace these values. You can use the <code>scripts/generate-hm9000-certs</code> script in the cf-release repository to generate self signed certs.
    </td>
  </tr>
</table>

<%= partial '../common/additional_config' %>

##<a id='generate'></a>Step 3: Generate Your Manifest

<%= partial '../common/generic_manifest' %>

