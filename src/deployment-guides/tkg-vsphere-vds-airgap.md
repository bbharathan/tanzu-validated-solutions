# Deploy Tanzu Kubernetes Grid on an Air-Gapped Environment

VMware Tanzu Kubernetes Grid (TKG) provides a consistent, upstream-compatible, regional Kubernetes substrate that is ready for end-user workloads and ecosystem integrations.

An air-gap installation method is used when the Tanzu Kubernetes Grid bootstrapper and cluster nodes components are unable to connect to the Internet to download the installation binaries from the public [VMware Registry](https://projects.registry.vmware.com/) during Tanzu Kubernetes Grid installation or upgrades. 

The scope of this document is limited to providing Prerequisites and deployment of Online Host and bootstrap machine

**Important:** we can refer same deployment guides of Tanzu for Kubernetes Operations based on the environemnt , in Airgap the change is going to be setting up Online Host, Private Repository and updating the Managment cluster config file with reposiroty details. 

## Supported Component Matrix

The following table provides the component versions and interoperability matrix supported with the reference design:

|**Software Components**|**Version**|
| --- | --- |
|Tanzu Kubernetes Grid|2.1.0|
|VMware vSphere ESXi|7.0 U3 |
|VMware vCenter Server|7.0 U3 |
|NSX Advanced Load Balancer|22.1.2|



### Prerequisites

1. Harbor Image Registry
2. Configure Online Host
3. Bootstrap VM

## <a id=install-harbor> </a> Install Harbor Image Registry

Install the Harbor only if you don’t have any existing image repository in your environment. 

To install Harbor, deploy an operating system of your choice with the following hardware configuration:

- vCPU: 4
- Memory: 8 GB
- Storage (HDD): 160 GB

Copy the Harbor binary from the bootstrap VM to the Harbor VM. Follow the instructions provided in [Harbor Installation and Configuration](https://goharbor.io/docs/2.3.0/install-config/) to deploy and configure Harbor.


## <a id=install-harbor> </a> Configure Online Host

Online host is a Virtual/Physical Machine which we use to download files from VMware repository for TKG Installation

### Prerequisites


   * A minimum of 6 GB of RAM , 2-core CPU , 160GB Disk.
   * System time is synchronized with a Network Time Protocol (NTP) server.
   * Docker and containerd binaries are installed. For instructions on how to install Docker, see [Docker documentation](https://docs.docker.com/engine/install/centos/).
   * Online Host should have access Internet Access.

To install Tanzu CLI, Tanzu Plugins, and Kubectl utility on the bootstrap machine, follow the instructions below:

1. Download and unpack the following Linux CLI packages from [VMware Tanzu Kubernetes Grid Download Product page](https://customerconnect.vmware.com/downloads/info/slug/infrastructure_operations_management/vmware_tanzu_kubernetes_grid/2_x).

   * VMware Tanzu CLI 2.1.0 for Linux
   * kubectl cluster cli v1.24.9 for Linux

1. Execute the following commands to install Tanzu Kubernetes Grid CLI, kubectl CLIs, and Carvel tools.
    ```bash
    ## Install required packages
    tdnf install tar zip unzip wget -y

    ## Install Tanzu Kubernetes Grid CLI
    tar -xvf tanzu-cli-bundle-linux-amd64.tar.gz
    cd ./cli/
    sudo install core/v0.28.0/tanzu-core-linux_amd64 /usr/local/bin/tanzu
    chmod +x /usr/local/bin/tanzu

    ## Verify Tanzu CLI version

     [root@tkg160-bootstrap ~] # tanzu version

    version: v0.28.0
    buildDate: 2023-01-20
    sha: 3c34115bc-dirty

    ## Install Tanzu Kubernetes Grid CLI Plugins

    [root@tkg160-bootstrap ~] # tanzu plugin sync

    Checking for required plugins...
    Installing plugin 'login:v0.28.0'
    Installing plugin 'management-cluster:v0.28.0'
    Installing plugin 'package:v0.28.0'
    Installing plugin 'pinniped-auth:v0.28.0'
    Installing plugin 'secret:v0.28.0'
    Installing plugin 'telemetry:v0.28.0'
    Installing plugin 'isolated-cluster:v0.28.0'
    Successfully installed all required plugins
    ✔  Done

    ## Verify the plugins are installed

    [root@tkg160-bootstrap ~]# tanzu plugin list
    NAME                DESCRIPTION                                                        TARGET       DISCOVERY  VERSION  STATUS
    login               Login to the platform                                                          default    v0.28.0  installed
    management-cluster  Kubernetes management-cluster operations                           kubernetes  default    v0.28.0  installed
    package             Tanzu package management                                           kubernetes  default    v0.28.0  installed
    pinniped-auth       Pinniped authentication operations (usually not directly invoked)              default    v0.28.0  installed
    secret              Tanzu secret management                                            kubernetes  default    v0.28.0  installed
    telemetry           Configure cluster-wide telemetry settings                                      default    v0.28.0  installed
    isolated-cluster    isolated-cluster operations                                        kubernetes  default    v0.28.0  installed


    ## Install Kubectl CLI
    gunzip kubectl-linux-v1.24.9+vmware.1.gz
    mv kubectl-linux-v1.24.9+vmware.1 /usr/local/bin/kubectl && chmod +x /usr/local/bin/kubectl

    # Install Carvel tools

    ##Install ytt
    cd ./cli
    gunzip ytt-linux-amd64-v0.43.1+vmware.1.gz
    chmod ugo+x ytt-linux-amd64-v0.43.1+vmware.1 &&  mv ./ytt-linux-amd64-v0.43.1+vmware.1 /usr/local/bin/ytt

    ##Install kapp

    cd ./cli
    gunzip kapp-linux-amd64-v0.53.2+vmware.1.gz
    chmod ugo+x kapp-linux-amd64-v0.53.2+vmware.1 && mv ./kapp-linux-amd64-v0.53.2+vmware.1 /usr/local/bin/kapp

    ##Install kbld

    cd ./cli
    gunzip kbld-linux-amd64-v0.35.1+vmware.1.gz
    chmod ugo+x kbld-linux-amd64-v0.35.1+vmware.1 && mv ./kbld-linux-amd64-v0.35.1+vmware.1 /usr/local/bin/kbld

    ##Install impkg

    cd ./cli
    gunzip imgpkg-linux-amd64-v0.31.1+vmware.1.gz
    chmod ugo+x imgpkg-linux-amd64-v0.31.1+vmware.1 && mv ./imgpkg-linux-amd64-v0.31.1+vmware.1 /usr/local/bin/imgpkg
    ```

1. Validate Carvel tools installation using the following commands.

    ```bash
    ytt version
    kapp version
    kbld version
    imgpkg version
    ```

1. Install `yq`. `yq` is a lightweight and portable command-line YAML processor. `yq` uses `jq`-like syntax but works with YAML and JSON files.

    ```bash
    wget https://github.com/mikefarah/yq/releases/download/v4.24.5/yq_linux_amd64.tar.gz

    tar -xvf yq_linux_amd64.tar.gz && mv yq_linux_amd64 /usr/local/bin/yq
    ```

1. Install `kind`.

    ```bash
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
    chmod +x ./kind
    mv ./kind /usr/local/bin/kind
    ```

1. Execute the following commands to start the Docker service and enable it to start at boot. Photon OS has Docker installed by default.

    ```bash
    ## Check Docker service status
    systemctl status docker

    ## Start Docker Service
    systemctl start docker

    ## To start Docker Service at boot
    systemctl enable docker
    ```

1. Execute the following commands to ensure that the bootstrap machine uses [cgroup v1](https://man7.org/linux/man-pages/man7/cgroups.7.html).

    ```bash
    docker info | grep -i cgroup

    ## You should see the following
    Cgroup Driver: cgroupfs
    ```

All required packages are now installed and the required configurations are in place in the bootstrap virtual machine. The next step is to deploy the Tanzu Kubernetes Grid management cluster.


1. Download the Images to the Online Machine

   - Important: Before performing this step, ensure that the disk partition where you download the images has 45 GB of available space.

      ```bash
      tanzu isolated-cluster download-bundle --source-repo <SOURCE-REGISTRY> --tkg-version <TKG-VERSION> --ca-certificate <SECURITY-CERTIFICATE>
      ```
      
   * SOURCE-REGISTRY is the IP address or the hostname of the registry where the images are stored.
   * TKG-VERSION is the version of Tanzu Kubernetes Grid that you want to deploy in the proxied or air-gapped environment.
   * SECURITY-CERTIFICATE is the security certificate of the registry where the images are stored. To bypass the security certificate validation, use --insecure, instead of --ca-certificate. Both the strings are optional. If you do not specify any value, the system validates the default server security certificate.

   - The following is an example:

      ```bash
      tanzu isolated-cluster download-bundle --source-repo projects.registry.vmware.com/tkg --tkg-version v2.1.0
      ```

      Your Internet-restricted environment is now ready for you to deploy


## <a id=configure-bootstrap> </a> Deploy and Configure Bootstrap VM / Offline VM
We will be using Same VM to upload Images to Private Registry and Deploy Tanzu Kubernetes Grid Clusters.<p>
The deployment of the Tanzu Kubernetes Grid management and workload clusters is facilitated by setting up a bootstrap machine where you install the Tanzu CLI and Kubectl utilities which are used to create and manage the Tanzu Kubernetes Grid instance. This machine also keeps the Tanzu Kubernetes Grid and Kubernetes configuration files for your deployments. The bootstrap machine can be a laptop, host, or server running on Linux, macOS, or Windows that you deploy management and workload clusters from.

The bootstrap machine runs a local `kind` cluster when Tanzu Kubernetes Grid management cluster deployment is started. Once the `kind` cluster is fully initialized, the configuration is used to deploy the actual management cluster on the backend infrastructure. After the management cluster is fully configured, the local `kind` cluster is deleted and future configurations are performed with the Tanzu CLI.

For this deployment, a Photon-based virtual machine is used as the bootstrap machine. For information on how to configure a macOS or a Windows machine, see [Install the Tanzu CLI and Other Tools](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.6/vmware-tanzu-kubernetes-grid-16/GUID-install-cli.html).

### Prerequisites


   * A minimum of 6 GB of RAM , 2-core CPU, 160GB .
   * System time is synchronized with a Network Time Protocol (NTP) server.
   * Docker and containerd binaries are installed. For instructions on how to install Docker, see [Docker documentation](https://docs.docker.com/engine/install/centos/).
   * Harbor Certificate need to be save in the system and Docker authentication should be completed (we will be using this VM to upload files to Harbor registry)
   * Copy the Files to the Offline Machine

To install Tanzu CLI, Tanzu Plugins, and Kubectl utility on the bootstrap machine, follow the instructions below:

1. Download and unpack the following Linux CLI packages from [VMware Tanzu Kubernetes Grid Download Product page](https://customerconnect.vmware.com/downloads/info/slug/infrastructure_operations_management/vmware_tanzu_kubernetes_grid/2_x).

   * VMware Tanzu CLI 2.1.0 for Linux
   * kubectl cluster cli v1.24.9 for Linux
1. **Copy the Files to the Offline Machine** <p>
   Copy the following files to the offline machine, which is the bootstrap machine in the proxied or air-gapped environment, through a USB thumb drive or other storage medium:
   * The Image TAR files
   * The YAML files
1. Execute the following commands to install Tanzu Kubernetes Grid CLI, kubectl CLIs, and Carvel tools.
    ```bash
    ## Install required packages , Ignore if the pakages is already installed.
    
     tar, zip, unzip & wget

    ## Install Tanzu Kubernetes Grid CLI
    tar -xvf tanzu-cli-bundle-linux-amd64.tar.gz
    cd ./cli/
    sudo install core/v0.28.0/tanzu-core-linux_amd64 /usr/local/bin/tanzu
    chmod +x /usr/local/bin/tanzu

    ## Verify Tanzu CLI version

    [root@tkg160-bootstrap ~] # tanzu version

    version: v0.28.0
    buildDate: 2023-01-20
    sha: 3c34115bc-dirty

    ## Install the Isolated Cluster Plugin on the Offline Machine

    [root@tkg160-bootstrap ~] tanzu plugin install isolated-cluster --local standalone-plugins/
    
    ## Log in to the Private Registry on the Offline Machine
    ## If your private registry uses a self-signed certificate, save the CA certificate of the registry in "/etc/docker/certs.d/registry.example.com/ca.crt"
    docker login <URL>

    ## Upload the Images to the Offline Machine
    tanzu isolated-cluster upload-bundle --source-directory <SOURCE-DIRECTORY> --destination-repo <DESTINATION-REGISTRY> --ca-certificate <SECURITY-CERTIFICATE>

    * SOURCE-DIRECTORY is the path to the location where the image TAR files are stored.
    * DESTINATION-REGISTRY is the path to the private registry where the images will be hosted in the air-gapped environment.
    * SECURITY-CERTIFICATE is the security certificate of the private registry where the images will be hosted in the proxied or air-gapped environment. 
    
    Example:- tanzu isolated-cluster upload-bundle --source-directory ./ --destination-repo registry.example.com/library --ca-certificate /etc/docker/certs.d/registry.example.com/ca.crt

    ##Configure the environment variables.
    In an air-gap environment, if you run the tanzu init or tanzu plugin sync commands, the command hangs and times out after some time with the following error:


    [root@tkg160-bootstrap ~]# tanzu init
    Checking for required plugins...
    unable to list plugin from discovery 'default': error while processing package: failed to get resource files from discovery: Checking if image is bundle: Fetching image: Get"https://projects.registry.vmware.com/v2/": dial tcp 10.188.25.227:443: i/o timeout
    All required plugins are already installed and up-to-date
    ✔  successfully initialized CLI
    ✔  Done

    By default the Tanzu global config file, `config.yaml`, which gets created when you first run `tanzu init` command, points to the repository URL <https://projects.registry.vmware.com> to fetch the Tanzu plugins for installation. Because there is no Internet in the environment, the commands fails after some time.

    To ensure that Tanzu Kubernetes Grid always pulls images from the local private registry, run the Tanzu `export` command to add `TKG_CUSTOM_IMAGE_REPOSITORY` to the global Tanzu CLI configuration file, `~/.config/tanzu/config.yaml`. 

    If your image registry is configured with a self-signed CA certificate, set the following environment variables.

    export TKG_CUSTOM_IMAGE_REPOSITORY=custom-image-repository.io/yourproject

    export TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY=false

    export TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE=LS0t[...]tLS0tLQ==

    If your image registry is configured with a public signed CA certificate, set the following environment variables.

    export TKG_CUSTOM_IMAGE_REPOSITORY=custom-image-repository.io/yourproject
    export TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY=true
    
    Note:If we reboot the VM , this setting will go to default 
    

    ## Install Tanzu Kubernetes Grid CLI Plugins

    [root@tkg160-bootstrap ~] # tanzu plugin sync

    Checking for required plugins...
    Installing plugin 'login:v0.28.0'
    Installing plugin 'management-cluster:v0.28.0'
    Installing plugin 'package:v0.28.0'
    Installing plugin 'pinniped-auth:v0.28.0'
    Installing plugin 'secret:v0.28.0'
    Installing plugin 'telemetry:v0.28.0'
    Installing plugin 'isolated-cluster:v0.28.0'
    Successfully installed all required plugins
    ✔  Done

    ## Verify the plugins are installed

    [root@tkg160-bootstrap ~]# tanzu plugin list
    NAME                DESCRIPTION                                                        TARGET       DISCOVERY  VERSION  STATUS
    login               Login to the platform                                                          default    v0.28.0  installed
    management-cluster  Kubernetes management-cluster operations                           kubernetes  default    v0.28.0  installed
    package             Tanzu package management                                           kubernetes  default    v0.28.0  installed
    pinniped-auth       Pinniped authentication operations (usually not directly invoked)              default    v0.28.0  installed
    secret              Tanzu secret management                                            kubernetes  default    v0.28.0  installed
    telemetry           Configure cluster-wide telemetry settings                                      default    v0.28.0  installed
    isolated-cluster    isolated-cluster operations                                        kubernetes  default    v0.28.0  installed


    ## Install Kubectl CLI
    gunzip kubectl-linux-v1.24.9+vmware.1.gz
    mv kubectl-linux-v1.24.9+vmware.1 /usr/local/bin/kubectl && chmod +x /usr/local/bin/kubectl

    # Install Carvel tools

    ##Install ytt
    cd ./cli
    gunzip ytt-linux-amd64-v0.43.1+vmware.1.gz
    chmod ugo+x ytt-linux-amd64-v0.43.1+vmware.1 &&  mv ./ytt-linux-amd64-v0.43.1+vmware.1 /usr/local/bin/ytt

    ##Install kapp

    cd ./cli
    gunzip kapp-linux-amd64-v0.53.2+vmware.1.gz
    chmod ugo+x kapp-linux-amd64-v0.53.2+vmware.1 && mv ./kapp-linux-amd64-v0.53.2+vmware.1 /usr/local/bin/kapp

    ##Install kbld

    cd ./cli
    gunzip kbld-linux-amd64-v0.35.1+vmware.1.gz
    chmod ugo+x kbld-linux-amd64-v0.35.1+vmware.1 && mv ./kbld-linux-amd64-v0.35.1+vmware.1 /usr/local/bin/kbld

    ##Install impkg

    cd ./cli
    gunzip imgpkg-linux-amd64-v0.31.1+vmware.1.gz
    chmod ugo+x imgpkg-linux-amd64-v0.31.1+vmware.1 && mv ./imgpkg-linux-amd64-v0.31.1+vmware.1 /usr/local/bin/imgpkg
    ```

1. Validate Carvel tools installation using the following commands.

    ```bash
    ytt version
    kapp version
    kbld version
    imgpkg version
    ```

1. Install `yq`. `yq` is a lightweight and portable command-line YAML processor. `yq` uses `jq`-like syntax but works with YAML and JSON files.

    ```bash
    DownloadLink:- https://github.com/mikefarah/yq/releases/download/v4.24.5/yq_linux_amd64.tar.gz

    tar -xvf yq_linux_amd64.tar.gz && mv yq_linux_amd64 /usr/local/bin/yq
    ```

1. Install `kind`.

    ```bash
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
    chmod +x ./kind
    mv ./kind /usr/local/bin/kind
    ```

1. Execute the following commands to start the Docker service and enable it to start at boot. Photon OS has Docker installed by default.

    ```bash
    ## Check Docker service status
    systemctl status docker

    ## Start Docker Service
    systemctl start docker

    ## To start Docker Service at boot
    systemctl enable docker
    ```

1. Execute the following commands to ensure that the bootstrap machine uses [cgroup v1](https://man7.org/linux/man-pages/man7/cgroups.7.html).

    ```bash
    docker info | grep -i cgroup

    ## You should see the following
    Cgroup Driver: cgroupfs
    ```
1. Create an SSH key-pair.

    This is required for Tanzu CLI to connect to vSphere from the bootstrap machine.  The public key part of the generated key will be passed during the Tanzu Kubernetes Grid management cluster deployment.  

    ```bash
    ### Generate public/Private key pair.

    ssh-keygen -t rsa -b 4096 -C "email@example.com"

    ### Add the private key to the SSH agent running on your machine and enter the password you created in the previous step 

    ssh-add ~/.ssh/id_rsa 

    ### If the above command fails, execute "eval $(ssh-agent)" and then rerun the command.
    ```

    Make a note of the public key from the file **$home/.ssh/id_rsa.pub**. You need this while creating a config file for deploying the Tanzu Kubernetes Grid management cluster.

1. If your bootstrap machine runs Linux or Windows Subsystem for Linux, and it has a Linux kernel built after the May 2021 Linux security patch, for example Linux 5.11 and 5.12 with Fedora, run the following command.

   ```
    sudo sysctl net/netfilter/nf_conntrack_max=131072
   ```

All required packages are now installed and the required configurations are in place in the bootstrap virtual machine. The next step is to deploy the Tanzu Kubernetes Grid management cluster.


### Management Cluster Configuration Template

We can use the tradital way of creating templates for management cluster,once the template is created we manually need to add  include Image repository configuration to the configuration file.<p>Sample config file is given below.

**Important:** Image repository configuration is very importat details which will not be part of default config file when we are creating from TKG UI.<p>


```yaml
#! ---------------------------------------------------------------------
#! Basic cluster creation configuration
#! ---------------------------------------------------------------------

CLUSTER_NAME:
CLUSTER_PLAN: <dev/prod>
INFRASTRUCTURE_PROVIDER: vsphere
ENABLE_CEIP_PARTICIPATION: <true/false>
ENABLE_AUDIT_LOGGING: <true/false>
CLUSTER_CIDR: 100.96.0.0/11
SERVICE_CIDR: 100.64.0.0/13
# CAPBK_BOOTSTRAP_TOKEN_TTL: 30m

#! ---------------------------------------------------------------------
#! vSphere configuration
#! ---------------------------------------------------------------------

VSPHERE_SERVER:
VSPHERE_USERNAME:
VSPHERE_PASSWORD:
VSPHERE_DATACENTER:
VSPHERE_RESOURCE_POOL:
VSPHERE_DATASTORE:
VSPHERE_FOLDER:
VSPHERE_NETWORK: <tkg-management-network>
VSPHERE_CONTROL_PLANE_ENDPOINT: #Leave blank as VIP network is configured in NSX ALB and IPAM is configured with VIP network

# VSPHERE_TEMPLATE:

VSPHERE_SSH_AUTHORIZED_KEY:
VSPHERE_TLS_THUMBPRINT:
VSPHERE_INSECURE: <true/false>
DEPLOY_TKG_ON_VSPHERE7: true

#! ---------------------------------------------------------------------
#! Node configuration
#! ---------------------------------------------------------------------

# SIZE:
# CONTROLPLANE_SIZE:
# WORKER_SIZE:
# OS_NAME: ""
# OS_VERSION: ""
# OS_ARCH: ""
# VSPHERE_NUM_CPUS: 2
# VSPHERE_DISK_GIB: 40
# VSPHERE_MEM_MIB: 4096
# VSPHERE_CONTROL_PLANE_NUM_CPUS: 2
# VSPHERE_CONTROL_PLANE_DISK_GIB: 40
# VSPHERE_CONTROL_PLANE_MEM_MIB: 8192
# VSPHERE_WORKER_NUM_CPUS: 2
# VSPHERE_WORKER_DISK_GIB: 40
# VSPHERE_WORKER_MEM_MIB: 4096

#! ---------------------------------------------------------------------
#! NSX Advanced Load Balancer configuration
#! ---------------------------------------------------------------------

AVI_CA_DATA_B64: 
AVI_CLOUD_NAME: 
AVI_CONTROL_PLANE_HA_PROVIDER: <true/false>
AVI_CONTROL_PLANE_NETWORK: 
AVI_CONTROL_PLANE_NETWORK_CIDR: 
AVI_CONTROLLER: 
AVI_DATA_NETWORK: 
AVI_DATA_NETWORK_CIDR: 
AVI_ENABLE: <true/false>
AVI_LABELS: 
AVI_MANAGEMENT_CLUSTER_CONTROL_PLANE_VIP_NETWORK_CIDR: 
AVI_MANAGEMENT_CLUSTER_CONTROL_PLANE_VIP_NETWORK_NAME: 
AVI_MANAGEMENT_CLUSTER_SERVICE_ENGINE_GROUP: 
AVI_MANAGEMENT_CLUSTER_VIP_NETWORK_CIDR: 
AVI_MANAGEMENT_CLUSTER_VIP_NETWORK_NAME: 
AVI_PASSWORD: <base 64 encoded AVI password>
AVI_SERVICE_ENGINE_GROUP: 
AVI_USERNAME: 


#! ---------------------------------------------------------------------
#! Image repository configuration
#! ---------------------------------------------------------------------

TKG_CUSTOM_IMAGE_REPOSITORY: ""
TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY: false
TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE: "base"

#! ---------------------------------------------------------------------
#! Machine Health Check configuration
#! ---------------------------------------------------------------------

ENABLE_MHC:
# ENABLE_MHC_CONTROL_PLANE: <true/false>
# ENABLE_MHC_WORKER_NODE: <true/flase>

#! ---------------------------------------------------------------------
#! Identity management configuration
#! ---------------------------------------------------------------------

IDENTITY_MANAGEMENT_TYPE: "none"

#! Settings for IDENTITY_MANAGEMENT_TYPE: "oidc"
# CERT_DURATION: 2160h
# CERT_RENEW_BEFORE: 360h
# OIDC_IDENTITY_PROVIDER_CLIENT_ID:
# OIDC_IDENTITY_PROVIDER_CLIENT_SECRET:
# OIDC_IDENTITY_PROVIDER_GROUPS_CLAIM: groups
# OIDC_IDENTITY_PROVIDER_ISSUER_URL:
# OIDC_IDENTITY_PROVIDER_SCOPES: "email,profile,groups"
# OIDC_IDENTITY_PROVIDER_USERNAME_CLAIM: email

#! Settings for IDENTITY_MANAGEMENT_TYPE: "ldap"
# LDAP_BIND_DN:
# LDAP_BIND_PASSWORD:
# LDAP_HOST:
# LDAP_USER_SEARCH_BASE_DN:
# LDAP_USER_SEARCH_FILTER:
# LDAP_USER_SEARCH_USERNAME: userPrincipalName
# LDAP_USER_SEARCH_ID_ATTRIBUTE: DN
# LDAP_USER_SEARCH_EMAIL_ATTRIBUTE: DN
# LDAP_USER_SEARCH_NAME_ATTRIBUTE:
# LDAP_GROUP_SEARCH_BASE_DN:
# LDAP_GROUP_SEARCH_FILTER:
# LDAP_GROUP_SEARCH_USER_ATTRIBUTE: DN
# LDAP_GROUP_SEARCH_GROUP_ATTRIBUTE:
# LDAP_GROUP_SEARCH_NAME_ATTRIBUTE: cn
# LDAP_ROOT_CA_DATA_B64:
```

For a full list of configurable values and to learn more about the fields present in the template file, see [Tanzu Configuration File Variable Reference](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.6/vmware-tanzu-kubernetes-grid-16/GUID-tanzu-config-reference.html).

Create a file using the values provided in the template and save the file with a `.yaml` extension. See [Appendix Section](#supplemental-information) for a sample YAML file to use for deploying a management cluster. 

After you have created or updated the cluster configuration file, you can deploy a management cluster by running the `tanzu mc create --file CONFIG-FILE` command, where CONFIG-FILE is the name of the configuration file. Below is the sample config file for deploying the TKG Management cluster in an air-gapped environment. 

```yaml
#! ---------------------------------------------------------------------
#! Basic cluster creation configuration
#! ---------------------------------------------------------------------

CLUSTER_NAME: tkg160-mgmt-airgap
CLUSTER_PLAN: prod
INFRASTRUCTURE_PROVIDER: vsphere
ENABLE_CEIP_PARTICIPATION: "true"
ENABLE_AUDIT_LOGGING: "true"
CLUSTER_CIDR: 100.96.0.0/11
SERVICE_CIDR: 100.64.0.0/13
# CAPBK_BOOTSTRAP_TOKEN_TTL: 30m

#! ---------------------------------------------------------------------
#! vSphere configuration
#! ---------------------------------------------------------------------

VSPHERE_SERVER: vcenter.lab.vmw
VSPHERE_USERNAME: tkg-admin@vsphere.local
VSPHERE_PASSWORD: <encoded:Vk13YXJlMSE=>
VSPHERE_DATACENTER: /tkgm-internet-dc1
VSPHERE_RESOURCE_POOL: /tkgm-internet-dc1/host/tkgm-internet-c1/Resources/tkg-management-components
VSPHERE_DATASTORE: /tkgm-internet-dc1/datastore/vsanDatastore
VSPHERE_FOLDER: /tkgm-internet-dc1/vm/tkg-management-components
VSPHERE_NETWORK: /tkgm-internet-dc1/network/tkg_mgmt_pg
VSPHERE_CONTROL_PLANE_ENDPOINT: #Leave blank as VIP network is configured in NSX ALB and IPAM is configured with VIP network

# VSPHERE_TEMPLATE:

VSPHERE_SSH_AUTHORIZED_KEY: ssh-rsa AAAA[...]== email@example.com
VSPHERE_TLS_THUMBPRINT: DC:FA:81:1D:CA:08:21:AB:4E:15:BD:2B:AE:12:2C:6B:CA:65:49:B8
VSPHERE_INSECURE: "false"
DEPLOY_TKG_ON_VSPHERE7: true

#! ---------------------------------------------------------------------
#! Node configuration
#! ---------------------------------------------------------------------

OS_NAME: photon
OS_VERSION: "3"
OS_ARCH: amd64
VSPHERE_CONTROL_PLANE_NUM_CPUS: 2
VSPHERE_CONTROL_PLANE_DISK_GIB: 40
VSPHERE_CONTROL_PLANE_MEM_MIB: 8192
VSPHERE_WORKER_NUM_CPUS: 2
VSPHERE_WORKER_DISK_GIB: 40
VSPHERE_WORKER_MEM_MIB: 8192

#! ---------------------------------------------------------------------
#! NSX Advanced Load Balancer configuration
#! ---------------------------------------------------------------------

AVI_CA_DATA_B64: LS0t[...]tLS0tLQ==
AVI_CLOUD_NAME: tanzu-vcenter01
AVI_CONTROL_PLANE_HA_PROVIDER: "true"
AVI_CONTROL_PLANE_NETWORK: tkg_cluster_vip_pg
AVI_CONTROL_PLANE_NETWORK_CIDR: 172.16.80.0/24
AVI_CONTROLLER: alb-ha.lab.vmw
AVI_DATA_NETWORK: tkg_workload_vip_pg
AVI_DATA_NETWORK_CIDR: 172.16.70.0/24
AVI_ENABLE: "true"
AVI_LABELS: 
AVI_MANAGEMENT_CLUSTER_CONTROL_PLANE_VIP_NETWORK_CIDR: 172.16.80.0/24
AVI_MANAGEMENT_CLUSTER_CONTROL_PLANE_VIP_NETWORK_NAME: tkg_cluster_vip_pg
AVI_MANAGEMENT_CLUSTER_SERVICE_ENGINE_GROUP: tanzu-mgmt-segroup-01
AVI_MANAGEMENT_CLUSTER_VIP_NETWORK_CIDR: 172.16.50.0/24
AVI_MANAGEMENT_CLUSTER_VIP_NETWORK_NAME: tkg_mgmt_vip_pg
AVI_PASSWORD: <encoded:Vk13YXJlMSE=>
AVI_SERVICE_ENGINE_GROUP: tanzu-wkld-segroup-01
AVI_USERNAME: admin


#! ---------------------------------------------------------------------
#! Image repository configuration
#! ---------------------------------------------------------------------

TKG_CUSTOM_IMAGE_REPOSITORY: "harbor-sa.lab.vmw/tkg-160"
TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY: false
TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE: LS0t[...]tLS0tLQ==

#! ---------------------------------------------------------------------
#! Machine Health Check configuration
#! ---------------------------------------------------------------------

ENABLE_MHC: true

#! ---------------------------------------------------------------------
#! Identity management configuration
#! ---------------------------------------------------------------------

IDENTITY_MANAGEMENT_TYPE: "none"

#! ---------------------------------------------------------------------
```

The cluster deployment logs are streamed in the terminal when you run the `tanzu mc create` command. The first run of `tanzu mc create` takes longer than subsequent runs because it has to pull the required Docker images into the image store on your bootstrap machine. Subsequent runs do not require this step, and thus the process is faster.

While the cluster is being deployed, you will find that a virtual service is created in NSX Advanced Load Balancer and new service engines are deployed in vCenter by NSX Advanced Load Balancer. The service engines are mapped to the SE Group `tanzu-mgmt-segroup-01`​.

Now you can access the Tanzu Kubernetes Grid management cluster from the bootstrap machine and perform additional tasks such as verifying the management cluster health and deploying the workload clusters.

To get the status of the Tanzu Kubernetes Grid management cluster execute the following command:

```bash
tanzu management-cluster get
```

![TKG management cluster status](img/tkg-airgap-vsphere-deploy/mgmt-cluster-status.jpg)

To interact with the management cluster using the kubectl command, retrieve the management cluster `kubeconfig` and switch to the cluster context to run kubectl commands.

```bash
# tanzu mc kubeconfig get --admin
Credentials of cluster 'tkg160-mgmt-airgap' have been saved
You can now access the cluster by running 'kubectl config use-context tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap'


]# kubectl config use-context tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap
Switched to context "tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap".
```


The Tanzu Kubernetes Grid management cluster is successfully deployed. You can now proceed with configuring custom ADCs and creating shared services & workload clusters.
## <a id="workloadalb"> </a> Configure AKO Deployment Config (ADC) for Workload Clusters
Tanzu Kubernetes Grid v1.6.x management clusters with NSX Advanced Load Balancer are deployed with 2 AKODeploymentConfigs.

* `install-ako-for-management-cluster`: default configuration for management cluster
* `install-ako-for-all`:  default configuration for all workload clusters. By default, all the workload clusters reference this file for their virtual IP networks and service engine (SE) groups. This ADC configuration does not enable NSX L7 Ingress by default.

As per this Tanzu deployment, create 2 more ADCs:

* `tanzu-ako-for-shared`: Used by shared services cluster to deploy the virtual services in `TKG Mgmt SE Group` and  the loadbalancer applications in `TKG Management VIP Network`.

* `tanzu-ako-for-workload-L7-ingress`: Use this ADC only if you would like to enable NSX Advanced Load Balancer L7 ingress on workload cluster. Otherwise, leave the cluster labels empty to apply the network configuration from default ADC `install-ako-for-all`.

### <a id="sharedako"> </a> Configure AKO Deployment Config (ADC) for Shared Services Cluster

As per the defined architecture, shared services cluster uses the same control plane and data plane network as the management cluster. Shared services cluster control plane endpoint uses `TKG Cluster VIP Network`, application loadbalancing uses `TKG Management Data VIP network` and the virtual services are deployed in `tanzu-mgmt-segroup-01` SE group. This configuration is enforced by creating a custom AKODeploymentConfig (ADC) and applying the respective `AVI_LABELS` while deploying the shared services cluster.

The format of the AKODeploymentConfig YAML file is as follows.

<!-- /* cSpell:disable */ -->
```yaml
apiVersion: networking.tkg.tanzu.vmware.com/v1alpha1
kind: AKODeploymentConfig
metadata:
  finalizers:
     - ako-operator.networking.tkg.tanzu.vmware.com
  generation: 2
  name: <Unique name of AKODeploymentConfig>
spec:
  adminCredentialRef:
    name: nsx-alb-controller-credentials
    namespace: tkg-system-networking
  certificateAuthorityRef:
    name: nsx-alb-controller-ca
    namespace: tkg-system-networking
  cloudName: <NAME OF THE CLOUD in ALB>
  clusterSelector:
    matchLabels:
      <KEY>: <VALUE>
  controlPlaneNetwork:
    cidr: <TKG-Cluster-VIP-CIDR>
    Name: <TKG-Cluster-VIP-Network>
  controller: <NSX ALB CONTROLLER IP/FQDN>
  dataNetwork:
    cidr: <TKG-Mgmt-Data-VIP-CIDR>
    name: <TKG-Mgmt-Data-VIP-Name>
  extraConfigs:
   cniPlugin: antrea
   disableStaticRouteSync: true
   ingress:
      defaultIngressController: false
      disableIngressClass: true
      nodeNetworkList:
      - networkName: <TKG-Mgmt-Network>     
  serviceEngineGroup: <Mgmt-Cluster-SEG>

```
<!-- /* cSpell:enable */ -->

The sample AKODeploymentConfig with sample values in place is as follows. You should add the respective avi label `type=shared-services` while deploying shared services cluster to enforce this network configuration.

* cloud: ​`sfo01w01vc01​`
* service engine group: `sfo01m01segroup01`
* Control Plane network: `sfo01-w01-vds01-tkgclustervip`
* VIP/data network: `sfo01-w01-vds01-tkgmanagementvip`
* Node Network: `sfo01-w01-vds01-tkgmanagement`

<!-- /* cSpell:disable */ -->
```yaml
apiVersion: networking.tkg.tanzu.vmware.com/v1alpha1
kind: AKODeploymentConfig
metadata:
  finalizers:
  - ako-operator.networking.tkg.tanzu.vmware.com
  generation: 2
  labels:
  name: tanzu-ako-for-shared
spec:
  adminCredentialRef:
    name: avi-controller-credentials
    namespace: tkg-system-networking
  certificateAuthorityRef:
    name: avi-controller-ca
    namespace: tkg-system-networking
  cloudName: sfo01w01vc01​
  clusterSelector:
    matchLabels:
      type: shared-services
  controlPlaneNetwork:
    cidr: 172.16.80.0/24
    name: sfo01-w01-vds01-tkgclustervip
  controller: alb-ha.lab.vmw
  dataNetwork:
    cidr: 172.16.50.0/24
    name: sfo01-w01-vds01-tkgmanagementvip
  extraConfigs:
    cniPlugin: antrea
    disableStaticRouteSync: true
    ingress:
      defaultIngressController: false
      disableIngressClass: true
      nodeNetworkList:
      - networkName: sfo01-w01-vds01-tkgmanagement
  serviceEngineGroup: tanzu-mgmt-segroup-01
```
<!-- /* cSpell:enable */ -->

After you have the AKO configuration file ready, use the `kubectl` command to set the context to Tanzu Kubernetes Grid management cluster and create the ADC:

<!-- /* cSpell:disable */ -->
```
# kubectl config use-context tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap
Switched to context "tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap".


# kubectl apply -f ako-shared-services.yaml
akodeploymentconfig.networking.tkg.tanzu.vmware.com/tanzu-ako-for-shared created
```
<!-- /* cSpell:enable */ -->

Use the following command to list all AKODeploymentConfigs created under the management cluster:

<!-- /* cSpell:disable */ -->
```
# kubectl get adc
NAME                                 AGE
install-ako-for-all                  16m
install-ako-for-management-cluster   16m
tanzu-ako-for-shared                 13s
```
<!-- /* cSpell:enable */ -->

### <a id="l7workloadako"> </a> Configure AKODeploymentConfig (ADC) for Workload Cluster to Enable NSX ALB L7 Ingress with NodePortLocal Mode

VMware recommends using NSX Advanced Load Balancer L7 ingress with NodePortLocal mode for the L7 application load balancing. This is enabled by creating a custom ADC with ingress settings enabled, and then applying the AVI_LABEL while deploying the workload cluster.  

As per the defined architecture, workload cluster cluster control plane endpoint uses `TKG Cluster VIP Network`, application loadbalancing uses `TKG Workload Data VIP network` and the virtual services are deployed in `tanzu-wkld-segroup-01` SE group.

Below are the changes in ADC Ingress section when compare to the default ADC.

* **disableIngressClass**: set to `false` to enable NSX ALB L7 Ingress.

* **nodeNetworkList**: Provide the values for TKG workload network name and CIDR.

* **serviceType**:  L7 Ingress type, recommended to use `NodePortLocal`

* **shardVSSize**: Virtual service size

The format of the AKODeploymentConfig YAML file for enabling NSX ALB L7 Ingress is as follows.

<!-- /* cSpell:disable */ -->
```yaml
apiVersion: networking.tkg.tanzu.vmware.com/v1alpha1
kind: AKODeploymentConfig
metadata:
  name: <unique-name-for-adc>
spec:
  adminCredentialRef:
    name: avi-controller-credentials
    namespace: tkg-system-networking
  certificateAuthorityRef:
    name: avi-controller-ca
    namespace: tkg-system-networking
  cloudName: <cloud name configured in nsx alb>
  clusterSelector:
    matchLabels:
      <KEY>: <value>
  controller: <ALB-Controller-IP/FQDN>
  controlPlaneNetwork:
    cidr: <TKG-Cluster-VIP-Network-CIDR>
    name: <TKG-Cluster-VIP-Network-CIDR>
  dataNetwork:
    cidr: <TKG-Workload-VIP-network-CIDR>
    name: <TKG-Workload-VIP-network-CIDR>
  extraConfigs:
    cniPlugin: antrea
    disableStaticRouteSync: false                               # required
    ingress:
      disableIngressClass: false                                # required
      nodeNetworkList:                                          # required
        - networkName: <TKG-Workload-Network>
          cidrs:
            - <TKG-Workload-Network-CIDR>
      serviceType: NodePortLocal                                # required
      shardVSSize: MEDIUM                                       # required
  serviceEngineGroup: <Workload-Cluster-SEG>


```
<!-- /* cSpell:enable */ -->

The AKODeploymentConfig with sample values in place is as follows. You should add the respective avi label `workload-l7-enabled=true` while deploying shared services cluster to enforce this network configuration.

* cloud: ​`sfo01w01vc01​`
* service engine group: `sfo01w01segroup01`
* Control Plane network: `sfo01-w01-vds01-tkgclustervip`
* VIP/data network: `sfo01-w01-vds01-tkgworkload`
* Node Network: `sfo01-w01-vds01-tkgworkload`

<!-- /* cSpell:disable */ -->
```yaml
apiVersion: networking.tkg.tanzu.vmware.com/v1alpha1
kind: AKODeploymentConfig
metadata:
  name: tanzu-ako-for-workload-l7-ingress
spec:
  adminCredentialRef:
    name: avi-controller-credentials
    namespace: tkg-system-networking
  certificateAuthorityRef:
    name: avi-controller-ca
    namespace: tkg-system-networking
  cloudName: sfo01w01vc01
  clusterSelector:
    matchLabels:
      workload-l7-enabled: "true"
  controller: 172.16.10.10
  controlPlaneNetwork:
    cidr: 172.16.80.0/24
    name: sfo01-w01-vds01-tkgclustervip
  dataNetwork:
    cidr: 172.16.70.0/24
    name: sfo01-w01-vds01-tkgworkloadvip
  extraConfigs:
    cniPlugin: antrea
    disableStaticRouteSync: false                               # required
    ingress:
      disableIngressClass: false                                # required
      nodeNetworkList:                                          # required
        - networkName: sfo01-w01-vds01-tkgworkload
          cidrs:
            - 172.16.60.0/24
      serviceType: NodePortLocal                                # required
      shardVSSize: MEDIUM                                       # required
  serviceEngineGroup: tanzu-wkld-segroup-01
```
<!-- /* cSpell:enable */ -->

Use the `kubectl` command to set the context to Tanzu Kubernetes Grid management cluster and create the ADC:

<!-- /* cSpell:disable */ -->
```
# kubectl config use-context tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap
Switched to context "tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap".


# kubectl apply -f workload-adc-l7.yaml
akodeploymentconfig.networking.tkg.tanzu.vmware.com/tanzu-ako-for-workload-l7-ingress created
```
<!-- /* cSpell:enable */ -->

Use the following command to list all AKODeploymentConfig created under the management cluster:

<!-- /* cSpell:disable */ -->
```
# kubectl get adc
NAME                                 AGE
install-ako-for-all                  17m
install-ako-for-management-cluster   17m
tanzu-ako-for-shared                 79s
tanzu-ako-for-workload-l7-ingress    12s
```
<!-- /* cSpell:enable */ -->

Now that you have successfully created the AKO deployment config, you need to apply the cluster labels while deploying the workload clusters to enable NSX Advanced Load Balancer L7 Ingress with NodePortLocal mode.

## <a id=deploy-tkg-shared-services> </a> Deploy Tanzu Kubernetes Grid Shared Services Cluster

Each Tanzu Kubernetes Grid instance can have only one shared services cluster. Create a shared services cluster if you intend to deploy Harbor.  

The procedures for deploying a shared services cluster and workload cluster are almost the same. A key difference is that for the shared service cluster you add the `tanzu-services` label to the shared services cluster, as its cluster role. This label identifies the shared services cluster to the management cluster and workload clusters.  

Shared services cluster uses the custom ADC `tanzu-ako-for-shared` created earlier to apply the network settings similar to the management cluster. This is enforced by applying the AVI_LABEL `type:shared-services` while deploying the shared services cluster.

Deployment of the shared services cluster is done by creating a YAML file and invoking the `tanzu cluster create -f <file-name>` command. The YAML file used for shared services deployment is usually a bit smaller than the YAML used for the management cluster deployment because you don’t need to define the AVI fields except `AVI_CONTROL_PLANE_HA_PROVIDER` & `AVI_LABELS` in the YAML. 

**Note:- We need to Provide the Private registry details in  yaml file**

The following is a sample YAML for deploying a shared services cluster:

```yaml
CLUSTER_NAME: tkg160-shared-services-airgap
CLUSTER_PLAN: prod
INFRASTRUCTURE_PROVIDER: vsphere
ENABLE_CEIP_PARTICIPATION: "true"
ENABLE_AUDIT_LOGGING: "true"
CLUSTER_CIDR: 100.96.0.0/11
SERVICE_CIDR: 100.64.0.0/13
VSPHERE_SERVER: vcenter.lab.vmw
VSPHERE_USERNAME: tkg-admin@vsphere.local
VSPHERE_PASSWORD: <encoded:Vk13YXJlMSE=>
VSPHERE_DATACENTER: /tkgm-internet-dc1
VSPHERE_RESOURCE_POOL: /tkgm-internet-dc1/host/tkgm-internet-c1/Resources/tkg-sharedsvc-components
VSPHERE_DATASTORE: /tkgm-internet-dc1/datastore/vsanDatastore
VSPHERE_FOLDER: /tkgm-internet-dc1/vm/tkg-sharedsvc-components
VSPHERE_NETWORK: /tkgm-internet-dc1/network/tkg_mgmt_pg
VSPHERE_CONTROL_PLANE_ENDPOINT: #Leave blank as VIP network is configured in NSX ALB and IPAM is configured with VIP network
VSPHERE_SSH_AUTHORIZED_KEY: ssh-rsa AAAA[...]== email@example.com
VSPHERE_TLS_THUMBPRINT: DC:FA:81:1D:CA:08:21:AB:4E:15:BD:2B:AE:12:2C:6B:CA:65:49:B8
VSPHERE_INSECURE: "false"
OS_NAME: photon
OS_VERSION: "3"
OS_ARCH: amd64
VSPHERE_CONTROL_PLANE_NUM_CPUS: 2
VSPHERE_CONTROL_PLANE_DISK_GIB: 40
VSPHERE_CONTROL_PLANE_MEM_MIB: 8192
VSPHERE_WORKER_NUM_CPUS: 2
VSPHERE_WORKER_DISK_GIB: 40
VSPHERE_WORKER_MEM_MIB: 8192
AVI_CONTROL_PLANE_HA_PROVIDER: "true"
AVI_LABELS: |
    'type': 'shared-services'
TKG_CUSTOM_IMAGE_REPOSITORY: "harbor-sa.lab.vmw/tkg-160"
TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY: false
TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE: LS0t[...]tLS0tLQ==
ENABLE_MHC: true
IDENTITY_MANAGEMENT_TYPE: "none"
```

Cluster creation takes approximately 15-20 minutes to complete. Verify the health of the cluster and validate the cluster labels applied.

1. After the cluster deployment completes, connect to the Tanzu Management Cluster context and verify the cluster labels.
    <!-- /* cSpell:disable */ -->
     ```bash
     ## Connect to tkg management cluster

    # kubectl config use-context tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap
    Switched to context "tkg160-mgmt-airgap-admin@tkg160-mgmt-airgap".
    
    ## verify the shared service cluster creation

     # tanzu cluster list
    NAME                           NAMESPACE  STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES   PLAN  TKR
    tkg160-shared-services-airgap  default    running  3/3           3/3      v1.23.8+vmware.2  <none>  prod  v1.23.8---vmware.2-tkg.1

    ## Add the tanzu-services label to the shared services cluster as its cluster role. In the following command "tkg160-shared-services-airgap” is the name of the shared service cluster

    # kubectl label cluster.cluster.x-k8s.io/tkg160-shared-services-airgap cluster-role.tkg.tanzu.vmware.com/tanzu-services="" --overwrite=true
    cluster.cluster.x-k8s.io/tkg160-shared-services-airgap labeled


    ## Validate that the AVI_LABEL has been applied and cluster is using the tanzu-ako-for-shared ADC.

     # kubectl get cluster tkg160-shared-services-airgap --show-labels
    NAME                            PHASE         AGE   VERSION   LABELS
    tkg160-shared-services-airgap   Provisioned   49m             cluster-role.tkg.tanzu.vmware.com/tanzu-services=,networking.tkg.tanzu.vmware.com/avi=tanzu-ako-for-shared,tanzuKubernetesRelease=v1.23.8---vmware.2-tkg.1,tkg.tanzu.vmware.com/cluster-name=tkg160-shared-services-airgap,type=shared-services
      ```
    <!-- /* cSpell:enable */ -->

1.  Connect to the admin context of the shared service cluster using the following commands and verify the ako pod status.
    <!-- /* cSpell:disable */ -->
     ```bash
    ## Use the following command to get the admin context of Shared Service Cluster. In the following command tkg-160-shared-svc is the name of the shared service cluster

    # tanzu cluster kubeconfig get tkg160-shared-services-airgap --admin
    Credentials of cluster 'tkg160-shared-services-airgap' have been saved
    You can now access the cluster by running 'kubectl config use-context tkg160-shared-services-airgap-admin@tkg160-shared-services-airgap'



    ## Use the following command to use the context of Shared Service Cluster

    # kubectl config use-context tkg160-shared-services-airgap-admin@tkg160-shared-services-airgap
    Switched to context "tkg160-shared-services-airgap-admin@tkg160-shared-services-airgap".

    
    # Verify that ako pod gets deployed in avi-system namespace

     kubectl get pods -n avi-system
    NAME    READY   STATUS    RESTARTS   AGE
    ako-0   1/1     Running   0          8m55s
     ```
    <!-- /* cSpell:enable */ -->
    
The shared services cluster is now successfully created, and you can proceed to deploying the workload clusters. 

## <a id=deploy-workload-cluster> </a> Deploy Tanzu Kubernetes Clusters (Workload Clusters)

The steps for deploying a workload cluster are almost exactly the same as for a shared services cluster, except that the the AVI Lables, names of the cluster and the placement containers (resource pools, VM folder, network, etc.) are different.

## <a id=deploy-packages> </a> Deploy User-Managed Packages

User-managed packages are installed after workload cluster creation. These packages extend the core functionality of Kubernetes clusters created by Tanzu Kubernetes Grid. 

Tanzu Kubernetes Grid includes the following user-managed packages. These packages provide in-cluster and shared services to the Kubernetes clusters that are running in your Tanzu Kubernetes Grid environment.

|**Function**|**Package**|**Location**|
| --- | --- | --- |
|Certificate Management|Cert Manager|Workload and shared services cluster|
|Container networking|Multus|Workload cluster|
|Container registry|Harbor|Shared services cluster|
|Ingress control|Contour|Workload and shared services cluster|
|Log forwarding|Fluent Bit|Workload cluster|
|Monitoring|Grafana<br>Prometheus|Workload cluster|

User-managed packages can be installed via CLI by invoking the `tanzu package install` command. Before installing the user-managed packages, ensure that you have switched to the context of the cluster where you want to install the packages.

Also, ensure that the tanzu-standard repository is configured on the cluster where you want to install the packages. By default, the newly deployed clusters should have the tanzu-standard repository configured.

You can run the command `tanzu package repository list -n tanzu-package-repo-global` to verify this. Also, ensure that the repository status is `Reconcile succeeded`.

```bash
 # Add Private Registry to the workload Cluster 

tanzu package repository add tanzu-standard --url harbor.tanzu.lab/tanzu-170/packages/standard/repo -n tkg-system

]# tanzu package repository list -A

NAME            REPOSITORY                                        TAG     STATUS               DETAILS
tanzu-standard  harbor.tanzu.lab/tanzu-170/packages/standard/repo        Reconcile succeeded
```



### Install Cert Manager

The first package that you should install on your cluster is the [**cert-manager**](https://github.com/cert-manager/cert-manager) package which adds certificates and certificate issuers as resource types in Kubernetes clusters and simplifies the process of obtaining, renewing and using those certificates.

1. Capture the available Cert Manager package versions.

    ```bash
    # tanzu package available list cert-manager.tanzu.vmware.com -A
    
    NAME                           VERSION               RELEASED-AT
    cert-manager.tanzu.vmware.com  1.5.3+vmware.7-tkg.1  2021-08-23 18:00:00 +0000 UTC
    cert-manager.tanzu.vmware.com  1.7.2+vmware.1-tkg.1  2021-10-29 18:00:00 +0000 UTC
    cert-manager.tanzu.vmware.com  1.7.2+vmware.3-tkg.1  2021-10-29 17:22:51 +0000 UTC
    ```


2. Install the `cert-manager` package.

    Capture the latest version from the previous command, if there are multiple versions available check the "RELEASED-AT" to collect the version of the latest one. This document make use of version 1.7.2+vmware.1-tkg.1 for installation.

    The following command installs the `cert-manager` package:

    ```bash
    tanzu package install cert-manager --package-name cert-manager.tanzu.vmware.com --namespace package-cert-manager --version <AVAILABLE-PACKAGE-VERSION> --create-namespace
    
    ]# tanzu package install cert-manager --package-name cert-manager.tanzu.vmware.com --namespace cert-manager-package --version 1.7.2+vmware.3-tkg.1 --create-namespace

    7:28:29AM: Deploy succeeded
    ```

1. Confirm that the `cert-manager` package has been installed successfully and the status is `Reconcile succeeded`.

    ```bash
   ]# tanzu package installed get cert-manager -n cert-manager-package
    NAME:                    cert-manager
    PACKAGE-NAME:            cert-manager.tanzu.vmware.com
    PACKAGE-VERSION:         1.7.2+vmware.3-tkg.1
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```

### Install Contour

[Contour](https://projectcontour.io/) is an open-source Kubernetes ingress controller providing the control plane for the Envoy edge and service proxy. Tanzu Kubernetes Grid includes signed binaries for Contour and Envoy, which you can deploy into workload clusters to provide ingress control services in those clusters.

After you have set up the cluster, you must first create the configuration file that is used when you install the Contour package and then install the package.

Package installation can be customized by entering the user-configurable values in YAML format. Following is an example YAML for customizing Contour installation.

```yaml
---
infrastructure_provider: vsphere
namespace: tanzu-system-ingress
contour:
 configFileContents: {}
 useProxyProtocol: false
 replicas: 2
 pspNames: "vmware-system-restricted"
 logLevel: info
envoy:
 service:
   type: LoadBalancer
   annotations: {}
   nodePorts:
     http: null
     https: null
   externalTrafficPolicy: Cluster
   disableWait: false
 hostPorts:
   enable: true
   http: 80
   https: 443
 hostNetwork: false
 terminationGracePeriodSeconds: 300
 logLevel: info
 pspNames: null
certificates:
 duration: 8760h
 renewBefore: 360h
```

For a full list of user-configurable values, see [Configure the Contour Extension](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-index.html)

1. Capture the available Contour package versions.

    ```bash
    # tanzu package available list contour.tanzu.vmware.com -n tkg-system

    NAME                      VERSION                RELEASED-AT
    contour.tanzu.vmware.com  1.22.3+vmware.1-tkg.1  2022-12-12 18:00:00 +0000 UTC
    ```

    Capture the latest version from the previous command. If there are multiple versions available check the "RELEASED-AT" to collect the version of the latest one. This document make use of version 1.22.3+vmware.1-tkg.1 for installation.

1. Install the Contour package.

    ```bash
    tanzu package install contour --package-name contour.tanzu.vmware.com --version <AVAILABLE-PACKAGE-VERSION> --values-file <Path_to_contour-data-values.yaml_file> --namespace tanzu-system-contour --create-namespace
    
    # tanzu package install contour --package-name contour.tanzu.vmware.com --version 1.20.2+vmware.1-tkg.1 --values-file ./contour-data-values.yaml --namespace tanzu-system-ingress --create-namespace

    7:45:21AM: Deploy succeeded
    ```

3. Confirm that the Contour package has been installed and the status is `Reconcile succeeded`.

    ```bash
    # tanzu package installed get contour --namespace tanzu-system-ingress

    NAME:                    contour
    PACKAGE-NAME:            contour.tanzu.vmware.com
    PACKAGE-VERSION:         1.22.3+vmware.1-tkg.1
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```

### Install Harbor

[Harbor](https://goharbor.io/) is an open-source container registry. Harbor Registry may be used as a private registry for container images that you want to deploy to Tanzu Kubernetes clusters.

Tanzu Kubernetes Grid includes signed binaries for Harbor, which you can deploy into:

- A workload cluster to provide container registry services for that clusters
- A shared services cluster to provide container registry services for other Tanzu Kubernetes (workload) clusters.

When deployed as a shared service, Harbor is available to all of the workload clusters in a given Tanzu Kubernetes Grid instance.

Follow this procedure to deploy Harbor into a workload cluster or a shared services cluster.

1. Confirm that the Harbor package is available in the cluster and retrieve the version of the available package.

    ```bash
    # tanzu package available list harbor.tanzu.vmware.com -A

    - Retrieving package versions for harbor.tanzu.vmware.com...

    NAME                     VERSION               RELEASED-AT                    NAMESPACE
    harbor.tanzu.vmware.com  2.3.3+vmware.1-tkg.1  2021-09-28 18:00:00 +0000 UTC  tanzu-package-repo-global
    harbor.tanzu.vmware.com  2.5.3+vmware.1-tkg.1  2021-09-28 18:00:00 +0000 UTC  tanzu-package-repo-global
    harbor.tanzu.vmware.com  2.6.3+vmware.1-tkg.1  2021-09-28 06:05:00 +0000 UTC  tanzu-package-repo-global
    ```



1. Create a configuration file named `harbor-data-values.yaml` by executing the following commands:

    ```bash
    image_url=$(kubectl -n tkg-system get packages harbor.tanzu.vmware.com.2.6.3+vmware.1-tkg.1 -o jsonpath='{.spec.template.spec.fetch[0].imgpkgBundle.image}')
    
    imgpkg pull -b $image_url -o /tmp/harbor-package --registry-ca-cert-path /etc/docker/certs.d/harbor.tanzu.lab/ca.crt

    cp /tmp/harbor-package/config/values.yaml harbor-data-values.yaml
    ```

1. Set the mandatory passwords and secrets in the `harbor-data-values.yaml` file

    ```bash
    bash /tmp/harbor-package/config/scripts/generate-passwords.sh harbor-data-values.yaml
    ```

1. Edit the `harbor-data-values.yaml` file and configure the values for the following mandatory parameters. 

      - namespace
      - port
      - harborAdminPassword
      - secretKey

    You can also change the values for other parameters to meet the requirements for your deployment. For the full list of the user-configurable values, see [Deploy Harbor into a Cluster](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-index.html).

1. Remove the comments in the `harbor-data-values.yaml` file

    ```bash
    yq -i eval '... comments=""' harbor-data-values.yaml
    ```

1. Install the Harbor package by executing the following command:

    ```bash
    # kubectl create namespace tanzu-system-registry
    # kubectl create namespace tanzu-harbor-registry
    # tanzu package install harbor --package harbor.tanzu.vmware.com --version 2.6.3+vmware.1-tkg.1 --values-file harbor-data-values.yaml --namespace tanzu-harbor-registry

    8:04:52AM: Deploy succeeded
    ```

7. Confirm that the Harbor package has been installed and the status is `Reconcile succeeded`.

    ```bash
    # tanzu package installed get harbor --namespace tanzu-system-registry

    NAME:                    harbor
    PACKAGE-NAME:            harbor.tanzu.vmware.com
    PACKAGE-VERSION:         2.6.3+vmware.1-tkg.1
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```

### Install Prometheus

[Prometheus](https://prometheus.io/) is a system and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true. Alertmanager handles alerts generated by Prometheus and routes them to their receiving endpoints.

Do the following to deploy Prometheus into a workload cluster:

1. Capture the available Prometheus version.

    ```bash
    # tanzu package available list prometheus.tanzu.vmware.com -A

    NAME                         VERSION                RELEASED-AT
    prometheus.tanzu.vmware.com  2.27.0+vmware.2-tkg.1  2021-05-12 18:00:00 +0000 UTC
    prometheus.tanzu.vmware.com  2.36.2+vmware.1-tkg.1  2022-06-23 18:00:00 +0000 UTC
    prometheus.tanzu.vmware.com  2.37.0+vmware.1-tkg.1  2022-10-25 18:00:00 +0000 UTC
    ```


    Capture the latest version from the previous command. If there are multiple versions available check the "RELEASED-AT" to collect the version of the latest one. This document make use of version  2.37.0+vmware.1-tkg.1 for installation.

2. Retrieve the template of the Prometheus package’s default configuration:

    ```bash
    image_url=$(kubectl -n tkg-system get packages prometheus.tanzu.vmware.com.2.37.0+vmware.1-tkg.1 -o jsonpath='{.spec.template.spec.fetch[0].imgpkgBundle.image}')

    imgpkg pull -b $image_url -o /tmp/prometheus-package-2.37.0+vmware.1-tkg.1 --registry-ca-cert-path /etc/docker/certs.d/harbor.tanzu.lab/ca.crt

    cp /tmp/prometheus-package-2.37.0+vmware.1-tkg.1/config/values.yaml prometheus-data-values.yaml
    ```

    This creates a configuration file named `prometheus-data-values.yaml` that you can modify.

1. To customize the Prometheus installation, modify the following values.

    |**Key**|**Default Value**|**Modified value**|
    | --- | --- | --- |
    |Ingress.tlsCertificate.tls.crt|Null|<p><Full chain cert provided in Input file></p><p></p><p>Note: This is optional.</p>|
    |ingress.tlsCertificate.tls.key|Null|<p><Cert Key provided in Input file</p><p></p><p>Note: This is optional.</p>|
    |ingress.enabled|false|true|
    |ingress.virtual_host_fqdn|prometheus.system.tanzu|prometheus."your-domain"|

    To see a full list of user configurable configuration parameters, see [Prometheus Package Configuration Parameters](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-index.html).

1. After you make any changes needed to your `prometheus-data-values.yaml` file, remove all comments in the file:

    ```bash
    yq -i eval '... comments=""' prometheus-data-values.yaml
    ```

1. Install Prometheus package.

    ```bash
    # kubectl create namespace tanzu-system-monitoring
    # kubectl create namespace tanzu-prometheus-monitoring
    # tanzu package install prometheus --package prometheus.tanzu.vmware.com --version 2.37.0+vmware.1-tkg.1--values-file prometheus-data-values.yaml --namespace tanzu-prometheus-monitoring

    8:04:52AM: Deploy succeeded
    ```

1. Confirm that the Prometheus package has been installed successfully and the status is 

    ```bash
    # tanzu package installed get prometheus -n tanzu-system-monitoring

    NAME:                    prometheus
    PACKAGE-NAME:            prometheus.tanzu.vmware.com
    PACKAGE-VERSION:         2.36.2+vmware.1-tkg.1
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```

### Install Grafana

[Grafana](https://grafana.com/) allows you to query, visualize, alert on, and explore metrics no matter where they are stored. Grafana provides tools to form graphs and visualizations from application data. 

**Note:** Grafana is configured with Prometheus as a default data source. If you have customized the Prometheus deployment namespace and it is not deployed in the default namespace, `tanzu-system-monitoring`, you need to change the Grafana data source configuration in the following code.

1. Retrieve the version of the available package.

    ```bash
    # tanzu package available list grafana.tanzu.vmware.com -A

    NAME                      VERSION                RELEASED-AT                    NAMESPACE
    grafana.tanzu.vmware.com  7.5.16+vmware.1-tkg.1  2022-05-19 18:00:00 +0000 UTC  tanzu-package-repo-global
    grafana.tanzu.vmware.com  7.5.16+vmware.1-tkg.2  2022-05-19 18:00:00 +0000 UTC  tanzu-package-repo-global
    grafana.tanzu.vmware.com  7.5.7+vmware.2-tkg.1   2021-05-19 18:00:00 +0000 UTC  tanzu-package-repo-global
    ```

    Capture the latest version from the previous command. If there are multiple versions available check the "RELEASED-AT" to collect the version of the latest one. This document make use of version 7.5.16+vmware.1-tkg.1 for installation.

2. Retrieve the template of the Grafana package’s default configuration.

    ```bash
    image_url=$(kubectl -n tanzu-package-repo-global get packages grafana.tanzu.vmware.com.7.5.16+vmware.1-tkg.1 -o jsonpath='{.spec.template.spec.fetch[0].imgpkgBundle.image}')

    imgpkg pull -b $image_url -o /tmp/grafana-package-7.5.16+vmware.1-tkg.1

    cp /tmp/grafana-package-7.5.16+vmware.1-tkg.1/config/values.yaml grafana-data-values.yaml
    ```

    This creates a configuration file named `grafana-data-values.yaml` that you can modify. For a full list of user-configurable values, see [Grafana Package Configuration Parameters](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/using-tkg-21/workload-packages-grafana.html).

4. Edit grafana-data-values.yaml and replace the following with your custom values.

    |**Key**|**Default Value**|**Modified value**|
    | --- | --- | --- |
    |grafana.service.type	|LoadBalancer|NodePort|
    |secret.admin_password|Null|Your password in Base64 encoded format.|
    |ingress.virtual_host_fqdn|grafana.system.tanzu	|User-Provided FQDN from Input File|
    |ingress.tlsCertificate.tls.crt	|Null|Full chain cert provided in Input file|
    |ingress.tlsCertificate.tls.key	|Null|Full chain cert provided in Input file|

5. (Optional) Modify the Grafana data source configuration. 

    Grafana is configured with Prometheus as a default data source. If you have customized the Prometheus deployment namespace and it is not deployed in the default namespace, `tanzu-system-monitoring`, you need to change the Grafana data source configuration in `grafana-data-values.yaml`.

    ```yaml
    datasources:
            - name: Prometheus
              type: prometheus
              url: prometheus-server.<change-to-prometheus-namespace>.svc.cluster.local
    ```

6. Remove all comments from `grafana-data-values.yaml` file

    ```bash
    yq -i eval '... comments=""' grafana-data-values.yaml
    ```

7. Install Grafana.

    ```bash
    # kubectl create namespace tanzu-system-dashboards
    # kubectl create namespace tanzu-grafana-dashboards
    # tanzu package install grafana --package grafana.tanzu.vmware.com --version 7.5.16+vmware.1-tkg.1 --values-file grafana-data-values.yaml --namespace tanzu-grafana-dashboards

    8:14:31AM: Deploy succeeded
    ```

1. Confirm that the Grafana package has been installed and the status is `Reconcile succeeded`.

    ```bash
    # tanzu package installed get grafana -n tanzu-system-dashboards

    NAME:                    grafana
    PACKAGE-NAME:            grafana.tanzu.vmware.com
    PACKAGE-VERSION:         7.5.16+vmware.1-tkg.1
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```

### Install Fluent Bit

[Fluent Bit](https://fluentbit.io/) is a lightweight log processor and forwarder that allows you to collect data and logs from different sources, unify them, and send them to multiple destinations.

The current release of Fluent Bit allows you to gather logs from management clusters or Tanzu Kubernetes clusters running in vSphere, Amazon EC2, and Azure. You can then forward them to a log storage provider such as [Elastic Search](https://www.elastic.co/), [Kafka](https://www.confluent.io/confluent-operator/), [Splunk](https://www.splunk.com/), or an HTTP endpoint. 

The example shown in this document uses HTTP endpoint `vRealize Log Insight` for forwarding logs from Tanzu Kubernetes clusters.

1. Retrieve the version of the available package.

    ```bash
    # tanzu package available list fluent-bit.tanzu.vmware.com -A

    NAME                         VERSION                RELEASED-AT                    NAMESPACE
    fluent-bit.tanzu.vmware.com  1.7.5+vmware.2-tkg.1   2021-05-13 18:00:00 +0000 UTC  tanzu-package-repo-global
    fluent-bit.tanzu.vmware.com  1.8.15+vmware.1-tkg.1   2022-05-24 18:00:00 +0000 UTC  tanzu-package-repo-global
    fluent-bit.tanzu.vmware.com  1.9.5+vmware.1-tkg.1  2022-06-23 18:00:00 +0000 UTC  tanzu-package-repo-global
    ```



    Capture the latest version from the previous command. If there are multiple versions available check the "RELEASED-AT" to collect the version of the latest one. This document make use of version 1.9.5+vmware.1-tkg.1 for installation.

2.  Retrieve the template of the Fluent Bit package’s default configuration.

    ```bash
    image_url=$(kubectl -n tanzu-package-repo-global get packages fluent-bit.tanzu.vmware.com.1.9.5+vmware.1-tkg.1 -o jsonpath='{.spec.template.spec.fetch[0].imgpkgBundle.image}')

    imgpkg pull -b $image_url -o /tmp/fluent-bit-1.8.15+vmware.1-tkg.1

    cp /tmp/fluent-bit-1.8.15+vmware.1-tkg.1/config/values.yaml fluentbit-data-values.yaml
    ```

3. Modify the resulting `fluentbit-data-values.yaml` file and configure the endpoint as per your choice. A sample endpoint configuration for sending logs to vRealize Log Insight Cloud over HTTP is shown in the following example.

    ```bash
    [OUTPUT]
            Name                 syslog
            Match                *
            Host                 vrli.lab.vmw
            Port                 514
            Mode                 udp
            Syslog_Format        rfc5424
            Syslog_Hostname_key  tkg_cluster
            Syslog_Appname_key   pod_name
            Syslog_Procid_key    container_name
            Syslog_Message_key   message
            Syslog_SD_key        k8s
            Syslog_SD_key        labels
            Syslog_SD_key        annotations
            Syslog_SD_key        tkg
    ```

4. Deploy Fluent Bit.

    ```bash
    # tanzu package available list fluent-bit.tanzu.vmware.com -n tkg-system
    # kubectl create namespace tanzu-system-logging
    # kubectl create namespace tanzu-fluent-bit-logging
    # tanzu package install fluent-bit --package fluent-bit.tanzu.vmware.com --version 1.9.5+vmware.1-tkg.1  --values-file grafana-data-values.yaml --namespace tanzu-fluent-bit-logging

    'PackageInstall' resource install status: Reconciling
    'PackageInstall' resource install status: ReconcileSucceeded

    Added installed package 'fluent-bit'
    ```

5. Confirm that the Fluent Bit package has been installed and the status is `Reconcile succeeded`.

    ```bash
    # tanzu package installed get fluent-bit -n tanzu-system-logging

    NAME:                    fluent-bit
    PACKAGE-NAME:            fluent-bit.tanzu.vmware.com
    PACKAGE-VERSION:         1.8.15+vmware.1-tkg.1
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```
