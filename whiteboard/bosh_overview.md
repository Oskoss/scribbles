# Simplifying Complex Systems with BOSH |

### Deploying Software (The Old Way)
- You have an awesome software product that will transform the fitness industry
- Have to decide where to run your product -- AWS it sounds trendy!

 ![Which cloud to choose?](resources/which-cloud.png "Which cloud to choose?")
- Start out running 1 VM - not too bad, easy enough to manage.
- App continues to mature pulling in more services which then requires more VMs.
- Still manageable

 ![More Services on AWS](resources/svcs-on-aws.png "More services on AWS")

- Dev went well, we have all of our services and app is ready to launch!

 ![Too many services!](resources/hard-to-mng.png "Too many services!")
- Too hard to manage all these services!
  - AWS S3 for storage
  - AWS VPC / Route 53 for networking
  - AWS CLI to manage all this....just too hard.

### Updating Software (The Old Way)
- Which VM/OS will be need to be updated?
- Will all the services need to be updated or just a few?
- How can we update without a product outage?

 #### Security Patching brings more questions
- How am I supposed to know about the security vulnerability?
- Are my services using the affected code?

 __"I'm an IT operator not a app developer nor the security expert! *This is too hard!*"__

___
### There's a better way! __BOSH__ |
- Explicit Resource Definition
  - Tells us what exactly is being used and in what manner
- Reproducible
  - Follows the same steps time over time to produce what is needed
- Guaranteed Consistency
  - Always utilizes the same code for every deployment from down deep (OS) to up top (processes)
- Fast & Agile
  - Only updates what is required when it is required (Continuous Integration :+1:)

### BOSH At A Glance

#### Lets start with the core and build out

- __Stemcell__: Each VM BOSH creates for you starts with a Stemcell - Foundation
  - Consists of a Operating System and base libraries
  - Minimal/bare as possible
  - Ubuntu Trusty or CentOS 7.x

   ![Foundation](resources/foundation-stemcell.png "Foundation")

- __Release__: The src that is intended to run - Building Materials
  - Versioned
  - Properties/ Configs
  - Startup Scripts
  - Source Code
  - Binaries
  - Anything required to get the software product running

   ![Materials](resources/materials.png "Materials")

- __Manifest__: Instructions for running the product - Blue Prints
  - Controls how and what you want to run

   ![Blue Prints](resources/blue-prints.png "Blue Prints")

- __Deployment__: All together these make a deployment
  - The final result

   ![Building](resources/building.png "Building")

#### Simple
  - The configuration of the 15+ services we had earlier is not simplified into a single command `bosh deploy`

   ![BOSH Deployment](resources/bosh-deployment.png "BOSH Deployment")

## Going Further

#### What is BOSH made of? (Draw small boxes/lines diagram of below)
- Single VM - Which all of BOSH runs on
- Director - Core orchestrator of all BOSH components
- Postgres DB - Storing information about the deployment
- NATS - Pub/Sub messaging system to communicate to each VM for instructions and health monitoring
- Blobstore - Storing releases and stemcells
- CPI - Cloud Provider Interface - Layer which abstracts infrastructure differences from the rest of BOSH.
- DNS Server - PowerDNS used to provide resolution between VMs in a deployment
- Agent - Each VM in all deployments has a small Agent which listens for instructions from the Director and then carries them out.
- ***TODO*** Maybe others I have forgotten?

#### Step 0 - Get BOSHed

- A single BOSH environment consists of the Director and the deployments that it orchestrates. To start, we need to deploy the director.
- Determine the infrastructure that is correct for you:
  - AWS
  - Azure
  - OpenStack
  - vSphere
  - vCloud
  - SoftLayer
  - GCP
  - Local (Good for local dev/test/exploration)
- [bosh-it](https://bosh.io/docs/using-bosh-init.html) your director -- automagically included in the cli v2.
- Communicating with your BOSH director:
  - To instruct BOSH to perform tasks or get the status of current deployments we utilize a [CLI](https://bosh.io/docs/bosh-cli.html).
  - While there is an API to talk to the BOSH director, that is not recommended (ever changing).
  - The CLI and BOSH has recently over gone a complete makeover for its release 2.0
  - Old CLI is written in ruby, new is written in Golang, many other changes -- will cover these later.

#### Stemcell
- Problem:
  - Heart Bleed Bug Hits -- How to update all 15 instances on AWS maintaining service without performance hits?
- Solution -- Stemcells!
- Stemcells are:
  - versioned
  - IaaS specifc
  - a bare minimum Operating System image with a BOSH agent
  - offered in two flavors - Ubuntu trusty - CentOS 7.x
  - packaged as tarballs
- Stemcells allow:
  - for a consistent and reproducible base OS
  - versioning of changes to the base OS
  - reuse of OS images across VMs of different types
  - reuse of OS images across different IaaS


#### Step 1 - Upload the stemcell
- Determine the stemcell that is correct for the deployment:
  - Ubuntu Trusty:
    - AWS Xen-HVM
    - AWS Xen-HVM
    - Azure Hyper-V
    - Google KVM
    - OpenStack KVM
    - SoftLayer Xen
    - vCloud ESXi
    - vSphere ESXi
    - Bosh Lite Warden
  - CentOS 7.x:
    - AWS Xen-HVM
    - AWS Xen-HVM
    - Azure Hyper-V
    - Google KVM
    - OpenStack KVM
    - SoftLayer Xen
    - vCloud ESXi
    - vSphere ESXi
    - Bosh Lite Warden


- Using the BOSH cli:
  - If your BOSH director is off the internet download the release from the [bosh.io](http://bosh.io) website and then upload the tarball
    - ```bash
      $ curl -L -J -O https://bosh.io/d/stemcells/bosh-aws-xen-hvm-ubuntu-trusty-go_agent?v=3363.14
      $ bosh upload stemcell bosh-aws-xen-hvm-ubuntu-trusty-go_agent.tar.gz
      ```
  - otherwise just upload directly from the internet
    - ```bash
      $ bosh upload stemcell https://bosh.io/d/stemcells/bosh-aws-xen-hvm-ubuntu-trusty-go_agent
      ```


#### Release
- Problem:
  - Updating code is hard
  - Updating code without outages is hard
  - Knowing what to update and what not to update is hard
- Solution -- Releases!
- Releases are:
  - versioned
  - layer placed on top of the stemcell
  - contain
    - configuration properties
    - configuration templates
    - start up scripts
    - source code
    - binaries
    - anything else required to build and deploy your product
- Releases allow:
  - recording of all dependencies needed for a product
  - versioning of product releases
  - products to be run on any IaaS, being IaaS agnostic
  - updating product throughout the whole deployment VM-by-VM
  - self-containment and no need for internet access for deployment

#### Step 2 - Creating a Release:
- Use the BOSH cli to build out the basic release structure
  - ```bash
    $ bosh init release fitAwesome
    ```

  - ![BOSH Release Tree](resources/bosh-release-tree.png "BOSH Release Tree")

    - blobs -> objects the product source code requires to run (I.E. in a Java world the JDK)
    - jobs -> All the units of work, think services and processes. Startup scripts, drain scripts, and configs.
    - packages -> All the packages the product relies on that need to be compiled for each VM. Remember BOSH can run the product on any IaaS and on CentOS or Ubuntu. Therefore we need to ensure all code can be run on both platforms.
    - src -> typical product source code you would distribute.
- Use the BOSH cli to create the packaged release
  - ```bash
    $ bosh create release
    ```
#### Step 3 - Upload the Release:
- Use the BOSH cli to upload the release to your director.
  - ```bash
    $ bosh upload release
    ```

#### Manifest
- Problem:
  - Separation code & configuration of such code is hard
  - Updating configuration without outages is hard
  - Knowing what is deployed and with what versions is hard
- Solution -- Manifest!
- Manifests are:
  - written in `YAML`
  - long
  - tedious
  - blueprints of your deployment
- Manifests allow:
  - explicit definition of resources used in deployment
  - explicit versioning of resources used in deployment
  - non-disruptive configuration changes of deployment
  - customer configuration of products in deployment

#### Step 4 - Creating a Manifest:
- Use your favorite text editor to create a `manifest.yml`
- Add the following to the manifest:
  - Deployment Identification: A name for the deployment and the UUID of the Director managing the deployment
  - Releases Block: Name and version of each release in a deployment
  - Networks Block: Network configuration information
  - Resource Pools Block: Properties of VMs that BOSH creates and manages
  - Disk Pools Block: Properties of disk pools that BOSH creates and manages
  - Compilation Block: Properties of compilation VMs
  - Update Block: Defines how BOSH updates job instances during deployment
  - Jobs Block: Configuration and resource information for jobs
  - Properties Block: Describes global properties and generalized configuration information

#### Step 5 - Deploy:
- Use the BOSH cli to start your deployment!
  - ```bash
    $ bosh deploy -d manifest.yml
    ```
## Learning Curve
- BOSH is powerful, with it comes a learning curve.

- ![BOSH Learning Curve](resources/bosh-learning.png "BOSH Learning Curve")

## BOSH Best Practices

#### Backing up BOSH
#### Bosh Backup command and(not or) IaaS snapshot disks
##### Dummy CPI
#### Multi-BOSH Deployment
#### Logging - quite verbose. use something to manage logs early (splunk)
#### Addons
#### Don't break the bosh pattern. Create a release and or a CPI
#### PowerDNS --> SPoF
#### BOSH Community Releases
#### BOSH 1.x vs 2.0
