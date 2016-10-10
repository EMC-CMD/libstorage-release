# libstorage-release
This repository is a [bosh release](https://bosh.io/releases) for [libstorage](https://github.com/emccode/libstorage).

##Prerequisites

The usage of this repository assumes that the developer have context using BOSH, BOSH CLI, and motiviation to use Libstorage for a storage solution.

This bosh release is open to use any adapter available libstorage client now or in the future. At the time of this writing, Libstorage supports VirtualBox, Isilon, and ScaleIO adapters for storage solutions. We will cover how to write a manifest for this release using Isilon and ScaleIO, and step by step deployment with that manifest.
__NOTE: Using two adapters in our manifest is for clear documentation. One could easily use only Isilon, or only ScaleIO with similar steps.__

####STEP 1: Clone & upload bosh release

Use the command `git clone https://github.com/EMC-CMD/libstorage-release.git` to clone this repository.

Create the release with the command `bosh create release --final`. This should add some release to the repository like `releases/libstorage-release/libstorage-release-1.yml`

Upload the bosh release provided with `bosh upload release releases/libstorage-release/libstorage-release-1.yml`. This should upload the release to your BOSH director.

####STEP 2: Create manifest

An example of this manifest is [here](example-manifest.yml). We will walk through chunks of this file, so it's recommended to copy+paste, then fix what's necessary for your configuration.

```
name: libstorage-example
director_uuid: <get-director-uuid>
```
The `name` should be fine, but you are welcome to change it to something more like `libstorage-job` or `libstorage`.
`director_uuid` needs to match the uuid of the bosh director we are trying to deploy to. To find this, use the BOSH CLI with `bosh status --uuid`.

```
releases:
- {name: libstorage, version: latest}
```
This line will specify what release your BOSH deployment will use. given that you should only have the deployment included in this repository, this line should be adequate.

```
jobs:
  - name: libstorage-example
    instances: 1
    networks:
    - name: my-favorite-network
    templates:
    - name: libstorage-example
      release: libstorage-example
    persistent_disk_pool: medium
    vm_type: medium
    stemcell: trusty
    azs:
    - my-az
```

The `jobs` block should contain information about the job BOSH will deploy. Give this job a cool `name` like `libstorage-is-the-best` or `libstorage-number-one`.

One `instances` might be enough, but this release supports scaling as needed. Note that hitting the endpoint for multiple servers would most likely require some proxy, and balancing. This task is outside the scope of the tutorial.

Assigning other values is the work of the developer with access to `bosh cloud-config` through the CLI. Fill in the necessary information, and be sure to specify a `stemcell` either in-line or as we have in our example by putting the alias `trusty` and a block at the bottom similar to the following:

```
stemcells:
- alias: trusty
  os: ubuntu-trusty
  version: '3262.9'
```

Don't forget the `update` block similar to what is shown below. For more details, see the [BOSH documentation](https://bosh.io/docs/deployment-manifest.html)

Important piece of this manifest is in `properties.libstorage`. This field is the configuration for your libstorage client. The client binary will use this configuration file at runtime. Use this as an example to guide structure for services you wish to provide through libstorage.
```
      properties:
        libstorage:
          libstorage:
            logging:
              level: debug
            client:
              type: controller
            host: tcp://0.0.0.0:7981 #This will listen for incoming connections to libstorage on this VM.
            server:
              services:
                isilonservice:
                  driver: isilon
                  isilon:
                    endpoint: https://<isilon-endpoint>:<isilon-port>
                    insecure: <true/false>
                    username: <a great username>
                    password: <a great password>
                    volumePath: <a path to mount volumes on this VM>
                    nfsHost: <nfs host for Isilon>
                    dataSubnet: <data subnet CIDR>
                    quotas: <true/false>
                scaleioservice:
                  driver: scaleio
                  scaleio:
                    endpoint: https://<scaleio endpoint>/api
                    insecure: <true/false>
                    userName: <secure name>
                    password: <secure password>
                    protectionDomainName: <protectionDomainName here>
                    storagePoolName: <storage pool name here>
                    version: <version goes here>
```

####Step 3: Target Manifest + Deploy

Use the commands:

`bosh deployment <path to created manifest>`
`bosh deploy`

To deploy libstorage. This may take some rinse-and-repeat with manifest options.

Congratulations! You've successfully deployed libstorage with BOSH.
