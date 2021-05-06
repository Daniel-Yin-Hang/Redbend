# **1. Overview**

This document describes Ignite thin installation for Vault.
Vault by HashiCorp is open source software for secret management.
Among the features that Vault provides:
- Secure, store, and tightly control access to tokens, passwords, and certificates
- Encryption keys for protecting secrets and other sensitive data
The Vault pod contains a containers server (handles all requests) and client (unseals Vault and initializes configuration at installation).
>>> Pic
# **2. Prerequisites**
**Ignite Thin installation process requires at least two servers:**

- Operator node: This machine runs the installation process only; it is not used by Ignite.
- Master: Ignite is installed on this machine. 
 
For **High Availability**, two additional nodes are required in the cluster (etcd quorum configuration).
>>> Pic
## **2.1 Operator Node**
The operator node is a machine outside the cluster.

This machine is used for installing and configuring the master and other nodes that are participating in the cluster.

Ignite Core is installed on top of Kubernetes using Docker, Helm, and more.

The components listed in this section should be installed on the operator node.

To check whether a component is already installed on the operator node, type the name of the component in the command line (for example, *helm*).

### **2.1.1 Helm**
**Helm** version 2.8.0 or higher.

Helm must be installed on the first cluster machine during the Ignite Core installation process and is the main deployment tool.

**To install Helm:**

*#cd /tmp/*
*#wget https://storage.googleapis.com/kubernetes-helm/helm-v2.8.0-linux-amd64.tar.gz*  
*#tar -zxvf helm-v2.8.0-linux-amd64.tar.gz*  
*#sudo mv linux-amd64/helm /usr/bin/*

### **2.1.2 kubectl**
Low-level management

**To install kubectl:**  
Follow the installation instructions in https://kubernetes.io/docs/tasks/tools/install-kubectl/

### **2.1.3 jq**
JSON parsing tool

**To install jq:**

*#sudo yum install -y jq.x86_64*  
Ubuntu: *#sudo apt-get install jq*

### **2.1.4 JMESPath**
Python module

**To install JMESPath:**

*#sudo yum install -y python-jmespath*  
Ubuntu: *#sudo apt-get install python-jmespath*

**If the** *python-jmespath* **installation fails with the error** "No package python-jmespath available", **install EPEL.**

*#rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm*

### **2.1.5 Ansible**
Version 2.4 or higher.

**To install Ansible:**  
Follow the installation instructions in the Ansible documentation.

## **2.2 Cluster – Master and Nodes**
The cluster is the machine that Ignite thin installation will be installed on.

The cluster can contain a master only, or a master and additional nodes for high availability.

### **2.2.1 Cluster Hardware Specification**
AWS model: t2.Small machine model
- OS: Red Hat 7.4
- RAM: 16 GB
- CPU: 2 cores
- Disk: 60 GB
### **2.2.2 Cluster Software Prerequisites**
iptables and any firewall should be disabled.

There are no other prerequisites.

You can use the *ilvault* image for the master and nodes.

## **2.3 Connection between Operator Node and Cluster**
The operator node should be able to access the cluster via SSH without asking for a password.
- For AWS: Use the AWS public key; for example, pem
- For non-AWS: Use the commands in the following sections to create and copy the private and public keys.

### **2.3.1 Generate SSH Private and Public Keys**
Run the following command:

*#ssh-keygen -t rsa*

Verify that **id_rsa.pub** and **id_rsa** were created by running the following command:

*#ls -latr ~/.ssh/*

### **2.3.2 Copy the Public Key to the Master and Nodes**
Run the following command and type the password when requested:

*#ssh-copy-id -i ~/.ssh/id_rsa.pub root@ignitesrv1.redbend.com*

## **2.4 Connecting Nodes to the Master**
If you are installing a master only, skip to section 3.  
1. Run the following command on the master and copy the token:  
*#cat /ignite/kubeadm/kubeadm-config.yaml** 
2. Run the following command on each node to add the node to the master:  
*#kubeadm join --token <token> --discovery-token-unsafe-skip-ca-verification <master IP>:6443*
3. Verify that all the nodes joined to the master by running the following command on the master:   
*#kubectl get nodes*

# **3. Preparation Steps on the Operator Node**
## **3.1 Prepare Installation Files**
To obtain the **install.tgz**, contact Customer Support.

Extract **ignite_install.tgz** to the operator node local directory.

## **3.2 Update Installation Files**
### **3.2.1 Update inventory.cfg**
Update **inventory.cfg** with the master and node IP addresses

If you are using a master only, leave the node empty

*#vi /tmp/ignite_install/ignite-environment-configuration/envs/thin-ignite-vault/spec/inventory.cfg*
### **3.2.2 Update kubernetes.yml**
Change the IP address and the host name to those of the master

*#vim /tmp/ignite_install/ignite-environment-configuration/envs/thin-ignite-vault/spec/kubernetes.yml*

** WSO2 is the Identity of the server used to manage user credentials.

- helm_repository
  - http://127.0.0.1:80 (change this only for AWS installation)
- load_balancer
  -  host_name: <master host name>
  -  user_portal_url: http://172.31.43.216:30080
  -  wso2_url: 172.31.43.216
  -  wso2_proxy_protocol: https
  -  wso2_port_number: 30010
  -  api_gateway_url: http://172.31.43.216:30080
  -  master: 172.31.43.216
- nfs
  - host_name: 172.31.43.216
- shared_content
  - nfs:
  - host_name: 172.31.43.216
  - path: /nfs
### **3.2.3 Update ignite-values.yml with your KMS Parameters (AWS only)**
*#vim /tmp/ignite_install/ignite-release-configuration/releases/2.11.0.0-latest/resources/ignite-values.yml*
- **For AWS: Update the Key Management Service (KMS) parameters for encrypted data**
- **For non-AWS: Update only the mode property**

The mode should be set according to:
- **prod: For production environment (using HTTPS)**
- **dev: For development environment (using HTTP)**
>**NOTE:** If you are using KMS, you must set mode to prod.  

Update the following values with your keys in the vault section:  
- mode: prod/dev
- export_unseal_data_to_file: true
- aws_env: true
- aws_access_key_id:
- aws_secret_access_key:
- aws_region_name: eu-central-1
- aws_kms_key_id:
### **3.2.4 Update release-manifest.yml**
Delete all sections except etcd and vault in the latest release version

*#vim /tmp/ignite_install/ignite-release-configuration/releases/2.11.0.0-latest/release-manifest.yml*
### **3.2.5 Delete post-install from all Versions**
(For example, 2.10.0.0 or 2.5.1.5)

*#cd /tmp/ignite_install/ignite-release-configuration/jobs/profile/development/installation/2.10.0.0*  
*#rm -rf post-install*
### **3.2.6 Delete 2-copy-wso2-keystore.yaml.j2 from the Latest Release Version**
*#cd /tmp/ignite_install/ignite-release-configuration/jobs/profile/development/installation/2.11.0.0/pre-install  
#rm -rf 2-copy-wso2-keystore.yaml.j2*
### **3.2.7 Update install.yml**
*#vim /tmp/ignite_install/ignite-operations/commands/ignite/install.yml*

The following actions should be removed by deleting their section:
- Scale identity server to match number of nodes
- Platform system tests
### **3.2.8 Delete the Vault Directory (AWS reinstallation only)**
Delete the Vault directory (located at **/nfs/igniteShare**) from the master.
## **3.3 Save Parameters**
Save the environment name and path definition by running the following commands on the operator node:

*#cd /tmp/ignite_install/ignite-operations*

*#export IGNITE_ENV_NAME=thin-ignite-vault*

*#export IGNITE_ENVS_DIR=/tmp/ignite_install/ignite-environment-configuration*

# **4. Installation Steps**
>**NOTE:** All scripts described in this section should be run from the **ignite-operations** directory.

*#cd /tmp/ignite_install/ignite-operations*

The installation log is located at **cat /var/log/messages**
## **4.1 Clean Kubernetes (Only for reinstallation on the same machine)**
Commands
- **For AWS:** ./ig.sh kubernetes tear-down --private-key /tmp/swm.pem
- **For local:** ./ig.sh kubernetes tear-down --private-key /root/.ssh/id_rsa

Another option is to run a full tear down (Kubernetes and configuration files) by using **hard-tear-down** instead of **tear-down**
## **4.2 Install Kubernetes**
For a fresh installation only, run the following commands on the master and other nodes:

*#echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> /etc/bashrc  
#echo "source <(kubectl completion bash)" >> ~/.bashrc* 

Ignite uses Kubernetes as a container orchestrator.

**To install Kubernetes, run:**
- **For AWS:** *#./ig.sh kubernetes bring-up --private-key /tmp/ignite_install/swm.pem*
- **For local:** *#./ig.sh kubernetes bring-up --private-key /root/.ssh/id_rsa*

The following components are installed by running this script:
- **Ingress HTTP backend**
- **Ingress controller**
- **etcd**
- **API server:** The main configuration entry point, exposes the Kubernetes API and secures communication with a cluster
- **controller-manager**
- **kube-dns:** Kubernetes internal DNS server, automatically registered in all pods
- **kube-flannel-ds:** Networking based on CNI plugin (daemonset as the number of servers)
- **kube-proxy:** Exposes Kubernetes services (daemonset as the number of servers)
- **kube-scheduler:** For booting and shutting down pods
- **kubernetes-dashboard-UI:** Kubernetes (not used in Ignite)
- **tiller-deploy**

**To verify that the pods are running, run:**

*#kubectl get pods -a --all-namespaces*

>**NOTE:** If the Kubernetes installation fails, you must clean Kubernetes (as described in section 4.1) and only then reinstall Kubernetes

## **4.3 Install dev-addons**
- **For AWS:** ./ig.sh kubernetes install-dev-addons --private-key /tmp/ignite_install/swm.pem
- **For local:** ./ig.sh kubernetes install-dev-addons --private-key /root/.ssh/id_rsa
The following components are installed by running this script.
- MailHog
- metrics-server
- NFS  

**To verify that the pods are running, run:**

*#kubectl get pods -a --all-namespaces*
## **4.4 Install Ignite**
- **For AWS:** ./ig.sh ignite install --target-release <release version> --config-dir /tmp/ignite_install/ignite-release-configuration --private-key /tmp/ignite_install/swm.pem  
**For local:** ./ig.sh ignite install --target-release <release version> --config-dir /tmp/ignite_install/ignite-release-configuration --private-key /root/.ssh/id_rsa  
**For** <release_version>, use the directories located at **./ignite-release-configuration/releases**
# **5. Verification Steps**
## **5.1 Verify the Vault Installation**
**To verify that the Vault installation was successful:**
1.  Verify that the Vault Pod is Running:  
   *#kubectl get pods*  
   The following pods should be listed:
    - dev-addons-mailhog
    - etcd-0, etcd-1, etcd-2
    - nfs-client-provisioner
    - vault-app
2.  Scale Vault deployment to two Pods for High Availability:  
    - Run the following command:  
    *#kubectl scale deployment vault-app --replicas=2*  
    - Verify that Vault has two pods:  
    *#kubectl get pods*
## **5.2 Verify the Vault Logs**
### **5.2.1 Server Log**
 Run the following command and verify that you get the correct result:

 *#kubectl logs vault-app-65bfc6895c-zwdtm vault-server*

 Expected result
 > ==> Vault server configuration:  
 > <center>Api Address: http://127.0.0.1:8200</center>
 > <center>Cgo: disabled</center>
 > <center>Cluster Address: https://127.0.0.1:8201</center>
 > <center>Listener 1: tcp (addr: "0.0.0.0:8202", cluster address: "0.0.0.0:8203", max_request_duration: "1m30s", max_request_size: "33554432", tls: "enabled")</center>
 > <center>Listener 2: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")</center>
 > <center>Listener 3: tcp (addr: "0.0.0.0:8204", cluster address: "0.0.0.0:8205", max_request_duration: "1m30s", max_request_size: "33554432", tls: "enabled")</center>
 > <center>Log Level: (not set)</center>
 > <center>Mlock: supported: true, enabled: false</center>
 > <center>Storage: etcd (HA available)</center>
 > <center>Version: Vault v1.0.2</center>
 > <center>Version Sha: 37a1dc9c477c1c68c022d2084550f25bf20cac33</center>
> ==> Vault server started! Log data will stream in below:

### **5.2.2 Client Log**
Run the following command and verify that you get the correct result:

*#kubectl logs vault-app-65bfc6895c-zwdtm vault-client*

Expected result

>##########End of insert data to Vault ###################  
>###########ignite-secret-values.json file was removed ###################  
>########### Starting sleep ###################

### **5.2.3 Vault UI**
To access the Vault UI, use https://host-name:30204/ui/vault/auth

The token is saved in **/nfs/igniteShared/vault/data/unseal_data** on the master.

**IMPORTANT:** Use the token and then do not forget to delete the **unseal_data file** so that the token is not saved.

If Vault is sealed, use the unseal keys first to open Vault.

### **5.2.4 Vault Ports**
Port to access the UI: 30204

Internal port to access Vault from inside Kubernetes microservices: 8200

External port to access Vault from outside Kubernetes: 30204

Internal port between the Vault client and the server: 8202

# **6. AWS Authentication**
If you are configuring Vault to AWS, the *auth* method is required; execute the script in Configuring AWS Authentication.
# **7. Advanced Commands**
Each pod can be stopped, started, or deleted.  
**For example:**  
*#kubectl stop pod dev-addons-mailhog*  
You can see the volume, ports, and actions of each pod

**For example:**  
*#kubectl describe pod dev-addons-mailhog*  
You can see the log (sdout) of each pod

**For example:**  
*#kubectl logs dev-addons-mailhog*
