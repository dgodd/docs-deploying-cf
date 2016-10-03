---
title: Using OpenStack Swift as a Cloud Foundry Blobstore
owner: Release Integration
menu:
  main:
    Name: Using OpenStack Swift as a Cloud Foundry Blobstore
    identifier: deploying-cf/openstack/using_swift_blobstore
    parent: deploying-cf/openstack
---



## Introduction ##

The Cloud Controller stores user-uploaded applications, buildpacks, droplets, and application resources in a blobstore. Examples of blobstore providers  include the local file system (Local), AWS S3 (AWS), and OpenStack Swift (OpenStack). This topic describes how to configure the Cloud Controller blobstore to use OpenStack Swift for storage.

The files uploaded by users are stored in private buckets and secured against
unauthorized access.
If a Droplet Execution Agent (DEA) needs to access these files from the Cloud
Controller's blobstore, the Cloud Controller generates temporary URLs pointing
to the required files and provides them to the DEA.
The DEA can use the URLs to download the files, execute staging tasks, then deliver back the results. To help ensure data security, the generated URLs are valid for a limited amount of time.

## OpenStack Prerequisites ##

To use the temporary URL feature, the OpenStack user needs the **ResellerAdmin** role.
You must configure an [X-Account-Meta-Temp-URL-Key](http://docs.openstack.org/havana/config-reference/content/object-storage-tempurl.html) for your OpenStack account to enable temporary URL generation. From a terminal window, follow the steps below to configure an X-Account-Meta-Temp-URL-Key.

1. Replace TENANT, USER, and PASSWORD in the command below. Run this command to
retrieve an auth token.

    ```
    curl -s -H 'Content-type: application/json' -d '{"auth": {"tenantName":     "TENANT", "passwordCredentials":
        {"username": "USER", \"password": "PASSWORD"}}}' https://auth.example.com:5000/v2.0/tokens
            | python -mjson.tool
    ```

1. Replace YOUR-AUTH-TOKEN, ACCOUNT-META-TEMP-URL-KEY, and YOUR-TENANT-ID in the command below. Run this command to assign an Account-Meta-Temp-URL-Key to your OpenStack account.

    ```
    curl -i -X POST -H 'X-Auth-Token:YOUR-AUTH-TOKEN' -H 'X-Account-Meta-Temp-URL-Key:ACCOUNT-META-TEMP-URL-KEY' \
    https://swift.example.com/v1/AUTH_YOUR-TENANT-ID
    ```

## Configure BOSH Deployment Manifest ##

You must provide the OpenStack credentials in your BOSH deployment manifest.
The following manifest snippet shows the required entries in the properties section of the deployment manifest. This example uses the OpenStack fog provider.

```
...
properties:
  ...
  ccng:
    ...
    packages:
      app_package_directory_key: cc-packages
      fog_connection: &fog_connection
        provider: 'OpenStack'
        openstack_username: '<user>'
        openstack_api_key: '<password>'
        openstack_auth_url: 'https://auth.example.com:5000/v2.0/tokens'
        openstack_temp_url_key: '<account meta temp url key>'
    droplets:
      droplet_directory_key: cc-droplets
      fog_connection: *fog_connection
    resource_pool:
      resource_directory_key: cc-resources
      fog_connection: *fog_connection
    buildpacks:
      buildpack_directory_key: cc-buildpacks
      fog_connection: *fog_connection
```

## Links ##

* [OpenStack Swift Temporary URL Documentation](http://docs.openstack.org/trunk/config-reference/content/object-storage-tempurl.html)
* [Fog gem](http://fog.io/)
* [OpenStack fog provider](https://github.com/fog/fog/tree/master/lib/fog/openstack)
