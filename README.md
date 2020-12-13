[![build_deploy_test Actions Status](https://github.com/ODIM-Project/ODIM/workflows/build_deploy_test/badge.svg)](https://github.com/ODIM-Project/ODIM/actions)
[![build_unittest Actions Status](https://github.com/ODIM-Project/ODIM/workflows/build_unittest/badge.svg)](https://github.com/ODIM-Project/ODIM/actions)

# Table of contents

- [Deploying the resource aggregator for ODIM (ODIMRA)](#deploying-the-resource-aggregator-for-odim--odimra-)
  * [1. Setting up environment](#1-setting-up-environment)
    + [Preparing the odim-controller node](#preparing-the-odim-controller-node)
      - [Installing Docker](#installing-docker)
      - [Building Docker images](#building-docker-images)
      - [Deploying Kubernetes on the odim-controller node](#deploying-kubernetes-on-the-odim-controller-node)
    + [Copying Docker images to the master and worker nodes](#copying-docker-images-to-the-master-and-worker-nodes)
  * [2. Installing ODIMRA](#2-installing-odimra)
    + [Default user credentials for ODIMRA and GRF Plugin](#default-user-credentials-for-odimra-and-grf-plugin)
- [Modifying default configuration parameters for the resource aggregator](#modifying-default-configuration-parameters-for-the-resource-aggregator)
- [Updating the Kubernetes Configuration file](#updating-the-kubernetes-configuration-file)
  * [**Sample Config file**](#--sample-config-file--)
- [Setting proxy](#setting-proxy)
- [Setting up NTP client](#setting-up-ntp-client)
- [Configuring proxy for Docker](#configuring-proxy-for-docker)
- [Resetting Kubernetes](#resetting-kubernetes)
- [Removing a node from the Kubernetes cluster](#removing-a-node-from-the-kubernetes-cluster)
- [Uninstalling ODIMRA](#uninstalling-odimra)


   



# Deploying the resource aggregator for ODIM (ODIMRA)

## 1. Setting up environment

This procedure provides step-by-step instructions on how to deploy Docker and Kubernetes. You will require a minimum of two machines - one as a deployment node where odim-controller is installed and the others as master and worker nodes where ODIMRA is deployed. 

**Prerequisites**
-----------------
- Ensure that the odim-controller node has a minimum RAM of 8GB (8192MB), four cups, and 100GB of Hdd.

- Ensure that the master and worker nodes have a minimum RAM of 16GB (16384MB), four cups, and 150GB of Hdd.

- Ensure that the Internet is available on all the machines. If the systems are behind a corporate proxy or firewall, [set your proxy configuration on all the machines](#setting-proxy).  

- Ensure to create same non-root username and password on all the machines and login with the same.

- Ensure not to create `odimra` user during the installation of the VM.

- Ensure that the FQDN entry in the `/etc/hosts` file is same on all the machines.

- Ensure that time across the machines are in sync.

- Ensure to [set the NTP client on the machines](#setting-up-ntp-client) to sync with NTP server in your environment.



**Procedure**
--------------
1. Download and install `Ubuntu 18.04 LTS` on your machines.
    >   **NOTE:**  Before installation, configure your system IP to access the data center network.
2. [Prepare the odim-controller node](#preparing-the-odim-controller-node). 
3. [Deploy Kubernetes](#deploying-kubernetes-on-the-odim-controller-node).
4. [Copy Docker images to master and worker nodes](#copying-docker-images-to-the-master-and-worker-nodes).   

  
   
	   
### Preparing the odim-controller node
------------------------------------------

**Procedure:**


1. Run the following commands to install packages such as Python, Ansible, Java 7, and more on the odim-controller node:
   1. ```
      $ sudo apt update
      ```
   2. ```
      $ sudo apt-get install sshpass -y
      ```
   3. ```
      $ sudo apt-get install python3.8 -y
      ```
   4. ```
      $ sudo apt install python3-pip -y
      ```
   5. ```
      $ sudo apt install software-properties-common -y
      ```
   6. ```
      $ sudo -E apt-add-repository ppa:ansible/ansible
      ```
	  Press enter when prompted.
   7. ```
      $ sudo apt install openjdk-8-jre-headless -y
      ```
   8. ```
      $ python3 -m pip install --upgrade pip
      ```
   9. ```
      $ git clone https://github.hpe.com/Bruce/odim-controller.git
      ```
	  When prompted for username and password, enter your HPE GitHub username and the personal access token or the SSH key. To know how to generate a personal token, see [Generating personal token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token). 
   10. ```
	   $ cd odim-controller/kubespray
	   ```
	 
	   ```
       $ sudo -H pip3 install --proxy=http://web-proxy.corp.hpecorp.net:8080/ -r requirements.txt
       ```
   11. ```
       $ pip3 install pyyaml==5.3.1
       ```
  12. ```
      $ pip3 install pycryptodomex==3.9.7
      ```
  13. ```
      $ wget https://dl.google.com/go/go1.13.7.linux-amd64.tar.gz -P /var/tmp
      ```
  14. ```
      $ sudo tar -C /usr/local -xzf /var/tmp/go1.13.7.linux-amd64.tar.gz
      ```
  15. ```
      $ export PATH=$PATH:/usr/local/go/bin
      ```
  16. ```
      $ mkdir ${HOME}/BRUCE
      ```
  17. ```
      $ cd ${HOME}/BRUCE
      ```
  18. ```
      $ mkdir src bin pkg
      ```
  19. ```
     $ export GOPATH=${HOME}/BRUCE
     ```
  20. ```
      $ export GOBIN=$GOPATH/bin
      ```
  21. ```
      $ export GO111MODULE=on
      ```
  22. ```
      $ export GOROOT=/usr/local/go
      ```
  23. ```
      $ export PATH=$PATH:${GOROOT}/bin
      ```
  24. ```
      $ sudo apt-get remove golang-docker-credential-helpers
      ```
  25. ```
      $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
      ```
  26. ```
      $ chmod 700 get_helm.sh
      ```
  27. ```
      $ ./get_helm.sh
      ```
2. Create and encrypt password files: 
   a. Navigate to the `odim-controller/scripts` directory.
       ```
       $ cd odim-controller/scripts
       ```
   b. Run the following command:
      ```
      $ go build -ldflags "-s -w" -o odim-vault odim-vault.go
      ```
   c. Create a file called `odimVaultKeyFile` and open it to edit.
       ```
       $ sudo vi odimVaultKeyFile
       ```
   d. Enter a password and save.
   e. Create a file called `nodePasswordFile` and open it to edit.
       ```
       $ sudo vi nodePasswordFile
       ```
   f. Enter the login password of your machine and save.
   g. Run the following commands to encode and encrypt the passwords:
      ```
       $ sudo ./odim-vault -encode /home/<username>/odim-controller/scripts/odimVaultKeyFile
       ```
	   
	   ```
       $ sudo ./odim-vault -key odimVaultKeyFile -encrypt /home/<username>/odim-controller/scripts/nodePasswordFile
       ```
3. [Edit the Kubernetes configuration file](#updating-the-kubernetes-configuration-file).
4. Update Firmware version:
   a. Navigate to `odim-controller/helmcharts/grfplugin-config/templates`.
      ```
      $ cd odim-controller/helmcharts/grfplugin-config/templates
      ```
   b. Open the `configmaps.yaml` file to edit.
      ```
      $ sudo vim configmaps.yaml
      ```
   c. Change *FirmwareVersion* to *v1.0.0*.
5. [Install Docker](#installing-docker).    	   
6. [Build Docker images](#building-docker-images).


#### Installing Docker
 > **IMPORTANT:** This procedure installs only the community edition of Docker.

1. To install Docker, run the following commands:
   1.  ```
       $ sudo apt update
       ```
   2.  ```
       $ sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
       ```
   3.  ```
       $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        ```
   4. ```
      $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
      ```
   5. ```
      $ sudo apt update
      ```
   6. ```
      $ apt-cache policy docker-ce
       ```
      
      The following output is generated:
      ```
      docker-ce:
      Installed: (none)
      Candidate: 18.03.1~ce~3-0~ubuntu
      Version table:
      18.03.1~ce~3-0~ubuntu 500
      500 https://download.docker.com/linux/ubuntu bionic/stable
      amd64 Packages
      ```
     
      > **NOTE:** docker-ce is not installed, but the candidate for  installation is from the Docker repository for Ubuntu 18.04 (bionic).
     
    7. ```
       $ sudo apt install docker-ce -y
       ```
    8. ```
       $ sudo apt-get install docker-compose -y
       ```
        
       >  **NOTE:** To run the commands without sudo, add your username to the docker group using the following command:
        ```
         $ sudo usermod -aG docker ${USER}
        ```
2. Check the status of Docker:
      ```
    $ sudo systemctl status docker
      ```
 
   If Docker is active and running, the following output is generated:
   ```
   docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor
   preset: enabled)
   Active: active (running) since Thu 2018-07-05 15:08:39 UTC; 2min 55s
   ago
   Docs: https://docs.docker.com
   Main PID: 10096 (dockerd)
   Tasks: 16
   CGroup: /system.slice/docker.service
   +-10096 /usr/bin/dockerd -H fd://
   +-10113 docker-containerd --config /var/run/docker/containerd/
   containerd.toml
   ```
     
   >  **NOTE:** If your system is behind a corporate proxy, ensure to configure Docker to use proxy server and restart docker services. To know how to configure Docker proxy, see [Configuring Docker proxy](#configuring-proxy-for-docker).
						   
     
3. Restart the server.
      ```
     $ sudo init 6
     ```
      
   >  **NOTE:** To enable Docker service to start on reboot, run the following command:
   
       `$ sudo systemctl enable docker`
	   
	   

#### Building Docker images

**Procedure**
----------------

To build Docker images, do the following:
1. Run the following commands:
   a. ```
      $ git clone https://github.com/ODIM-Project/ODIM.git
      ```
   b. ```
      $ cd ODIM
      ```
   c. ```
      $ export ODIMRA_USER_ID=2021
	  ```
	  ```
      $ export ODIMRA_GROUP_ID=2021
	  ```
   d. ```
      $ ./build_images.sh
      ```
   e. ```
      $ sudo docker images
      ```
	  If the images are built successfully, you will receive an output which is similar to the following sample:
	 ```
	 REPOSITORY                                TAG                                      IMAGE ID                   CREATED                     SIZE
     consul                                    1.6                                       33ff2311df24               4 hours ago                  185MB
     odim_zookeeper                            1.0                                      981d43f6c8b4              22 hours ago                 278MB
     update                                    1.0                                      2cfb65430181              22 hours ago                 128MB
     task                                      1.0                                      c4dd52b9ade0              22 hours ago                129MB
     systems                                   1.0                                       9d3ad9845b16             22 hours ago                129MB
     redis                                     1.0                                      81bf552d3a52              22 hours ago                 99.2MB
     managers                                  1.0                                      2f7955586c54              22 hours ago                 128MB
     odim_kafka                                1.0                                     f03a52363483              22 hours ago                  278MB
     grf-plugin                                1.0                                     c7a086d02b16              22 hours ago                  100MB
     fabrics                                   1.0                                     9b9ea8bafc30                22 hours ago                  128MB
     events                                    1.0                                     860a9202a483              22 hours ago                  130MB
     api                                       1.0                                     effab530ede5                22 hours ago                  130MB
     aggregation                               1.0                                     354f67a857b6               22 hours ago                  130MB
     account-session                           1.0                                     a7eb07e69395              22 hours ago                  129MB
	 ```
2. Save each Docker image to a tar archive using the following command:
    ```
    $ sudo docker save -o <image_name.tar> <image_name:tag>
    ```
	Example: sudo docker save -o consul.tar consul:1.0
3. Copy each tar archive to the second machine using the following command:
    ```
    $ scp <image_name.tar> <username>@<IP_address>:/<path>
    ```
	

 
### Deploying Kubernetes on the odim-controller node

**Procedure**
-----------------
1. Navigate to `odim-controller/scripts` on the odim-controller node.
   ```
   cd odim-controller/scripts
   ```
2. Run the following command:
   ```
   python3 odim-controller.py --deploy kubernetes --config /home/<username>/odim-controller/scripts/kube_deploy_nodes.yaml
   ```
3. Log in to the master and worker nodes and run the following commands:
   ```
   $ mkdir -p $HOME/.kube
   ```
   
   ```
   $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   ```
   
   ```
   $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
4. To verify that the Kubernetes pods are up and running in the master and worker nodes, run the following command:
   ```
   $ kubectl get pods -n kube-system -o wide
   ```
   
   
### Copying Docker images to the master and worker nodes

**Procedure**
---------------
Do the following in the master and the worker nodes:
1. Log in and create a file called `load_images.sh` in the same path where the ODIMRA Docker image tarballs are copied.
   ```
    $ sudo vim load_images.sh
    ```
2. Copy the following lines into `load_images.sh` and save.
    ```
	#!/bin/bash

    images_list=( task managers grf-plugin events update systems fabrics api aggregation account-session redis consul odim_zookeeper odim_kafka)

    for image in "${images_list[@]}"; do
        docker load -i ${image}.tar
    done
    ```	
3. Run the following commands:
   ```
   $ chmod +x load_images.sh
   ```
   ```
   $ ./load_images.sh
   ```
4. Verify that Docker images are available using the following command:
   ```
   $ sudo docker images
   ```
   
## 2. Installing ODIMRA

**Procedure**
--------------
1. Log in to the odim-controller node.
2. Navigate to `odim-controller/scripts`.
   ```
   cd odim-controller/scripts
   ```
3. Run the following command:
   ```
   $ python3 odim-controller.py --deploy odimra --config /home/<username>/odim-controller/scripts/kube_deploy_nodes.yaml
   ```
4. Log in to the master node.
5. To verify that the odimra pods are up and running, run the following command:
   ```
   $ kubectl get pods -n odim -o wide
   ```
6. To add the Generic Redfish Plugin and servers to the resource aggregator for ODIM, refer to the following readme.  
    https://github.com/ODIM-Project/ODIM/blob/development/svc-aggregation/README.md
	
	
### Default user credentials for ODIMRA and GRF Plugin 


ODIMRA:

```
Username: admin
Password: Od!m12$4
```

GRF plugin:

```
Username: admin
Password: GRFPlug!n12$4
``` 
 
 
  
#  Modifying default configuration parameters for the resource aggregator

1.   Navigate to the `build_odimra_1` container using the following command: 

      ```
     $ docker exec -it build_odimra_1/bin/bash
     ```

2.   Edit the parameters in the `odimra_config.json` file located in this path: `/etc/odimra_config/odimra_config.json` and save. 

     The parameters that are configurable are listed in the following table.
      > **NOTE:** It is recommended not to modify parameters other than the ones listed in the following table.
     
     |Parameter|Type|Description|
     |---------|----|-----------|
     |RootServiceUUID|String|Static `UUID` used for the resource aggregator root service.  NOTE: Take a backup copy of `RootServiceUUID` as it is required during reinstallation.|
     |LocalhostFQDN|String|FQDN of the host.|
     |KeyCertConf{|Array| |
     |RootCACertificatePath|String|TLS Root CA file path (which can be a chain of CAs for verifying entities interacting with the resource aggregator services).|
     |RPCPrivateKeyPath|String|TLS private key file path for the microservice RPC communications.|
     |RPCCertificatePath}|String|TLS certificate file path for the microservice RPC communications.|
     |APIGatewayConf{|Array| |
     |Host|String|Host address for the resource aggregator API gateway.|
     |Port|String|Port for the resource aggregator API gateway.|
     |PrivateKeyPath|String|TLS private key file path for the API gateway.|
     |CertificatePath}|String|TLS certificate file path for the API gateway.|
     |TLSConf{|Array|TLS configuration parameters.<br> Note: It is not recommended to change these settings. |
     |MinVersion|String|Default value: `TLS1.2`<br> Supported values: `TLS1.0, TLS1.1, TLS1.2`<br> Recommended value: `TLS1.2`|
     |MaxVersion|String|Default value: `TLS1.2`<br> Supported values: `TLS1.0, TLS1.1, TLS1.2`<br>  Recommended value: `TLS1.2`<br>  NOTE: If `MinVersion` and `MaxVersion` are not specified, they will be set to default values.<br> If `MinVersion` and `MaxVersion` are set to unsupported values, the resource aggregator and the plugin services will exit with errors.|
     |VerifyPeer|Boolean| Default value: true<br>  Recommended value: true<br>  NOTE:<br>  - `VerifyPeer` is set to true, by default. For secure plugin interaction, add root CA certificate (that is used to sign the certificates of the southbound entities) to root CA certificate file.  If `VerifyPeer` is set to false, SSL communication will be insecure.  After setting `VerifyPeer` to false, restart the resource aggregator container (`odim_1`).<br>  - If `TLS1.2` is used, ensure that the entity certificate has `SAN` field for successful validation.  - Northbound entities interacting with resource aggregator `API` service can use root CA that signed odimra's certificate.|
     |PreferredCipherSuites}|List| Default and supported values: See "List of supported (default) cipher suites".<br> IMPORTANT:<br>  - If `PreferredCipherSuites` is not specified, it will be set to default cipher (secure) suites.<br>  - If `PreferredCipherSuites` is set to unsupported cipher suites, the resource aggregator and the plugin services will exit with errors.|

     List of supported (default) cipher suites:
     
     |||
     |----|----|
     |TLS_RSA_WITH_AES_128_GCM_SHA256|Supported in TLS1.2|
     |TLS_RSA_WITH_AES_256_GCM_SHA384|Supported in TLS1.2|
     |TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256|Supported in TLS1.2|
     |TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384|Supported in TLS1.2|
     |TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256|Supported in TLS1.2|
     |TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384|Supported in TLS1.2|

3.   Exit from Docker using the following command: 

     ```
     $ exit
     ```

4.   Restart Docker using the following command: 

     ```
     $ docker restart build_odimra_1
      ```
 
# Updating the Kubernetes Configuration file

**Procedure**
--------------
1. Navaigate to the `odim-controller/scripts` directory.
      ```
       $ cd odim-controller/scripts
       ```
2. Rename the `kube_deploy_nodes.yaml.tmpl` file to `kube_deploy_nodes.yaml` using the following command:
      ```
       $ mv kube_deploy_nodes.yaml.tmpl kube_deploy_nodes.yaml
       ```
3. Open the `kube_deploy_nodes.yaml` file to edit.
   
**Sample Config file**
----------------------
```
deploymentID: <Unique identifier for the deployment>
httpProxy: <HTTP Proxy to be set in the nodes>
httpsProxy: <HTTPS Proxy to be set in the nodes>
noProxy: <NO PROXY env to be set in the nodes>
nodePasswordFilePath: <Absolute path of the file containing encrypted node password>
nodes:
  <Node1_Hostname>:
    ip: <Node1_IP_Address>
    username: <Node1_Username>
  <Node2_Hostname>:
    ip: <Node2_IP_Address>
    username: <Node2_Username>
  <Node3_Hostname>:
    ip: <Node3_IP_Address>
    username: <Node3_Username>
odimControllerSrcPath: <Absolute_Path_odim-controller_Src>
odimVaultKeyFilePath: <Absolute path of the file containing encoded odim vault password>
odimra_cert_path: <Absolute_path_of_odimra_certificates>
 
odimra:
  groupID: 2021
  userID: 2021
  namespace: odim
  fqdn: ''
  hostIP: ''
  rootServiceUUID: ''
  redisInMemoryHost: redis-inmemory
  redisOnDiskHost: redis-ondisk
  redisHAEnabled: false
  connectionMethodConf:
  - ConnectionMethodType: Redfish
    ConnectionMethodVariant: Compute:BasicAuth:GRF_v1.0.0
  - ConnectionMethodType: Redfish
    ConnectionMethodVariant: Storage:BasicAuth:STG_v1.0.0
  kafkaHost: kafka
  kafkaPort: 9092
  grfPluginRootServiceUUID:
  grfPluginUsername: admin
  grfPluginPassword: "UUFCYFpBoHh6UdvytPzm65SkHj5zyl73EYVNJNbrFeAPWYrkpTijGB9zrVQSbbLv052HK7-7chqDQQcjgWf7YA=="
  grfPluginLBHost: ''
  grfPluginLBPort: 30081
  etcHostsEntries:
  appsLogPath: /var/log/odimra
 
  odimraServerCertFQDNSan: "<CSV of FQDNs to include in ODIM-RA server certificate SAN>"
  odimraServerCertIPSan: "<CSV of IPs to include in ODIM-RA server certificate SAN>"
  odimraKafkaClientCertFQDNSan: "<CSV of FQDNs to include in ODIM-RA kafka client certificate SAN>"
  odimraKafkaClientCertIPSan: "<CSV of IPs to include in ODIM-RA kafka client certificate SAN>"
 
  apiNodePort: 30080
  apiReplicaCount: 1
  accountSessionReplicaCount: 1
  aggregationReplicaCount: 1
  eventReplicaCount: 1
  fabricsReplicaCount: 1
  managersReplicaCount: 1
  systemsReplicaCount: 1
  taskReplicaCount: 1
  updateReplicaCount: 1
  pluginReplicaCount: 1
  pluginEventNodePort: 30081
  pluginLogPath: /var/log/grfplugin_logs
 
  consulDataPath: /etc/consul/data
  consulConfPath: /etc/consul/conf
 
  kafkaConfPath: /etc/kafka/conf
  kafkaDataPath: /etc/kafka/data
  kafkaJKSPassword: "K@fk@_store1"
 
  redisOndiskDataPath: /etc/redis/data/ondisk
  redisInmemoryDataPath: /etc/redis/data/inmemory
 
  zookeeperConfPath: /etc/zookeeper/conf
  zookeeperDataPath: /etc/zookeeper/data
  zookeeperJKSPassword: "K@fk@_store1"
 
  rootCACert:
  odimraServerCert:
  odimraServerKey:
  odimraRSAPublicKey:
  odimraRSAPrivateKey:
  odimraKafkaClientCert:
  odimraKafkaClientKey:

```

Update the following parameters in the `kube_deploy_nodes.yaml` file:

|Parameter|Description|
|---------|-----------|
|deploymentID| A unique identifier to identify the Kubernetes cluster. Example: *Kubenode*<br>`deploymentID` is required for the following operations:<ul><li>Adding a node.</li><li>Deleting a node.</li><li>Resetting the Kubernetes cluster.</li><li>Deploying and removing ODIMRA services.</li></ul>|
|httpProxy|HTTP Proxy to be used for connecting to external network.|
|httpsProxy|HTTPS Proxy to be used for connecting to external network.|
|noProxy|List of IP addresses and FQDNs for which proxy should not be used.<br> `noProxy` should begin with `127.0.0.1,localhost,localhost.localdomain,10.96.0.0/12,` followed by the IP addresses of the nodes.<br>Example:<ul><li>For one node setup:<br>`noProxy: "127.0.0.1,localhost,localhost.localdomain,10.96.0.0/12,<Node_IP>"`</li><li>For three node setup:<br>`noProxy: "127.0.0.1,localhost,localhost.localdomain,10.96.0.0/12,<Node1_IP>,<Node2_IP>,<Node3_IP>"`</li></ul> |
|nodePasswordFilePath|The absolute path of the file containing the node's encoded password - */home/<username>/odim-controller/scripts/nodePasswordFile*|
|nodes:|List of nodes that are part of the kubernetes cluster.<br> **Note:**For one node setup, only one node's information is required. Remove the entries of the other two nodes.|
|Node1_Hostname|Hostname of the master node. To know the hostname, run the following command:<br>`$ hostname`|
|ip|Ip address of the master node.|
|username|Username of the master node.<br>**Important:** Ensure that the username is same across all the nodes.|
|odimControllerSrcPath|The absolute path of the downloaded odim-controller source code - */home/<username>/odim-controller*|
|odimVaultKeyFilePath|The absolute path of the file containing odim-vault's encoded password - */home/<username>/odim-controller/scripts/odimVaultKeyFile*|
|odimra_cert_path| The absolute path of the directory where certificates required for ODIM-RA services are present.<br> `odimra_cert_path` will be updated to a default path when odim-controller generates certificates required for the ODIM-RA services. Optionally, you can update it to a different path.<br> **Important:** Before specify a different path, ensure that the directory contains all the required certificates.|
|odimra|List of configuration parameters required for deploying ODIMRA and third party services.|
|fqdn| FQDN of the master node where ODIMRA will be deployed.|
|hostIP|IP address of the master node where the Generic Redfish (GRF) plugin will be deployed.|
|rootServiceUUID|RootServiceUUID to be used by ODIMRA and the plugin service.|
|grfPluginRootServiceUUID|RootServiceUUID for the GRF plugin.|
|grfPluginLBHost|The host address of the LB server configured for receiving events from a compute server.<br> **Important:** `grfPluginLBHost` must be same as `hostIP`.|



   
# Setting proxy

**Procedure**
----------------
To set proxy:
1. Open the `/etc/environment' file.
   ```
   $ sudo vim /etc/environment
   ```
2. Add the following lines and save:
   ```
   http_proxy=http://16.85.88.10:8080/
   https_proxy=http://16.85.88.10:8080/
   no_proxy="127.0.0.1,localhost,localhost.localdomain,10.96.0.0/12,<Node1_IP>,<Node2_IP>"
   EOF
   ```
   <Node1_IP> is the IP address of the first machine.
   <Node2_IP> is the IP address of the second machine.
   
   **Important:** When you add a new node to the cluster, ensure to update no_proxy with the IP address of the new node.

  
# Setting up NTP client 

**Procedure**
---------------
To configure NTP client, do the following:
1. Run the following commands:
   a. ```
      $ sudo apt update -y
      ```
   b. ```
      $ sudo apt install ntpdate -y
      ```
   c. ```
      $ sudo timedatectl set-ntp off
      ```
   d. ```
      $ sudo apt install ntp -y 
      ```
2. Open the `/etc/ntp.conf` file to edit.
   ```
   $ sudo vim /etc/ntp.conf
   ```
3. Comment the following lines:
   > #pool 0.ubuntu.pool.ntp.org iburst
     #pool 1.ubuntu.pool.ntp.org iburst
     #pool 2.ubuntu.pool.ntp.org iburst
     #pool 3.ubuntu.pool.ntp.org iburst
4. Add the following line and save:
   ```
   server <NTP_server_IP_address> prefer iburst
   ``
5. Restart NTP server by running the following command:
   ```
   $ sudo systemctl restart ntp
   ``
6. To verify the time sync with the NTP server on the NTP client machine, run the following command:
   ```
   $ ntpq -pd
   ``
   
# Configuring proxy for Docker

<blockquote>
IMPORTANT:

During the course of this procedure, you will be required to create files and copy content into them. In the content to be copied, substitute the parameters listed in the following table with their original values:

|Parameter|Description|
|---------|-----------|
|`<Proxy_URL>` |Your company URL.|
|`<ODIM_server_VM_IP>` |The IP address of the system where the resource aggregator is installed.|
|`<FQDN>` |FQDN of the resource aggregator server.|

</blockquote>

**Procedure**
--------------

1.   In the home directory of odimra user, create a hidden directory called .docker, and then create a file called config.json inside it.

      ```
       mkdir .docker
      ```

      ```
       cd .docker
      ```

      ```
       vi config.json
      ```	   

2.   Add the following content in the config.json file and save: 

      ```
      {
         "proxies":
        {
           "default":
          {
             "httpProxy": "<Proxy_URL>",
             "httpsProxy": "<Proxy_URL>",
             "noProxy": "localhost,127.0.0.1, <ODIM_server_VM_IP>"
          }
        }
      }
    
     ```

3.   Update the `/etc/environment` file with the following content using sudo: 

      ```
      export http_proxy=<Proxy_URL>
      export https_proxy=<Proxy_URL>
      HOSTIP=<ODIM_server_VM_IP>
      FQDN=<FQDN>
      ```

4.   Do the following on the resource aggregator server: 

     1. Create a directory using the following command: 

         ```
         $ sudo mkdir /etc/systemd/system/docker.service.d/
         ```

     2. Create a file called http-proxy.conf in the `/etc/systemd/system/docker.service.d/` directory. 
     3. Add the following content in the `http-proxy.conf` file and save: 

         ```
         [Service]
         Environment="HTTP_PROXY=<Proxy_URL>"
         Environment="HTTPS_PROXY=<Proxy_URL>"
         Environment="NO_PROXY=localhost,127.0.0.1, <ODIM_server_VM_IP>"
        
         ```

     4. Run the following commands: 

         ```
         $ sudo systemctl daemon-reload
         ```

         ```
         $ sudo service docker restart
         ```


# Resetting Kubernetes
  
  To reset Kubernetes setup, run the following command:
  ```
  $ python3 odim-controller.py --reset odimra --config /home/<username>/odim-controller/scripts/kube_deploy_nodes.yaml
  ```
  
# Removing a node from the Kubernetes cluster

  To remove an existing node form the kubernetes cluster, run the following command:
  ```
  $ python3 odim-controller.py --rmnode kubernetes --config /home/<username>/odim-controller/scripts/kube_deploy_nodes.yaml
  ```

# Uninstalling ODIMRA
  
  To remove ODIMRA, run the following command:
  
  ```
  $ python3 odim-controller.py --reset odimra --config /home/<username>/odim-controller/scripts/kube_deploy_nodes.yaml
  ```
  
