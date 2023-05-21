# **MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER & DOCKER COMPOSE**

# **Step 1 - Install client tools before bootstrapping the cluster**

### Install and configure AWS CLI
```
aws configure --profile %your_username%
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

### Test your AWS CLI by running:
![verifying aws cli](../screenshots/project21/verifying_aws_cli.jpg)   
*Verifying aws cli*  
<br>
 
### Install kubectl
Kubernetes cluster has a Web API that can receive HTTP/HTTPS requests, but it is quite cumbersome to curl an API each and every time you need to send some command, so kubectl command tool was developed to ease a K8s administrator’s life.

```
#Download the binary
wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl

#Make it executable
chmod +x kubectl

#Move to the Bin directory
sudo mv kubectl /usr/local/bin/
```
![verifying kubectl](../screenshots/project21/verifying_kubectl.jpg)   
*Verifying kubectl*  
<br>

### Install CFSSL and CFSSLJSON
```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson

#Make it executable
chmod +x cfssl cfssljson

#Move to the Bin directory
sudo mv cfssl cfssljson /usr/local/bin/
```
![verifying CFSSL and CFSSLJSON](../screenshots/project21/verifying_CFSSL_CFSSLJSON.jpg)   
*Verifying CFSSL and CFSSLJSON*  
<br>

# **Step 2 - Configure network infrastructure**

### Virtual Private Cloud – VPC
Create a directory named k8s-cluster-from-ground-up
![create directory](../screenshots/project21/create_directory.jpg)   
*create directory*  
<br>


Create a VPC and store the ID as a variable:
![create VPC](../screenshots/project21/create_VPC.jpg)   
*Create VPC*  
<br>

Tag the VPC so that it is named:
![tag VPC](../screenshots/project21/tag_VPC.jpg)   
*Tag VPC*  
<br>

### Domain Name System – DNS
Enable DNS and hostname support for your VPC:
![enable DNS and hostname support](../screenshots/project21/enable_DNS_hostname_support1.jpg)   
*Enable DNS and hostname support*  
<br>

![enable DNS and hostname support](../screenshots/project21/enable_DNS_hostname_support2.jpg)   
*Enable DNS and hostname support*  
<br>

### AWS Region
Set the required region
![set region](../screenshots/project21/set_region.jpg)   
*Set region*  
<br>

### Dynamic Host Configuration Protocol – DHCP
Configure DHCP Options Set:
![configure set](../screenshots/project21/configure_set.jpg)   
*Configure set*  
<br>

Tag the DHCP Option set:
![tag set](../screenshots/project21/tag_set1.jpg)   
*Tag set*  
<br>

![tag set](../screenshots/project21/tag_set2.jpg)   
*Tag set*  
<br>

Associate the DHCP Option set with the VPC:
![associate set](../screenshots/project21/associate_set1.jpg)   
*Associate set*  
<br>

![associate set](../screenshots/project21/associate_set2.jpg)   
*Associate set*  
<br>

### Subnet
Create and tag the Subnet:
![create and tag subnet](../screenshots/project21/create_tag_subnet.jpg)   
*Create and tag subnet*  
<br>

### Internet Gateway – IGW
Create the Internet Gateway and attach it to the VPC:
![configure IGW](../screenshots/project21/configure_IGW.jpg)   
*Configure IGW*  
<br>

### Route tables
Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:
![configure route tables](../screenshots/project21/configure_route_tables.jpg)   
*Configure route tables*  
<br>

### Security Groups
Configure security groups
```
# Create the security group and store its ID in a variable
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name ${NAME} \
  --description "Kubernetes cluster security group" \
  --vpc-id ${VPC_ID} \
  --output text --query 'GroupId')

# Create the NAME tag for the security group
aws ec2 create-tags \
  --resources ${SECURITY_GROUP_ID} \
  --tags Key=Name,Value=${NAME}

# Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

# # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'

# Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 6443 \
  --cidr 0.0.0.0/0

# Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create ICMP ingress for all types
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol icmp \
  --port -1 \
  --cidr 0.0.0.0/0
```

![create security group](../screenshots/project21/create_sg.jpg)   
*Create security group*
<br>


### Network Load Balancer
Create a network Load balancer:
![create load balancer](../screenshots/project21/create_load_balancer1.jpg)   
*create load balancer*
<br>

![create load balancer](../screenshots/project21/create_load_balancer2.jpg)   
*create load balancer*
<br>

### Tagret Group
Create a target group: (For now it will be unhealthy because there are no real targets yet.)
![create target group](../screenshots/project21/create_target_group1.jpg)   
*Create target group*
<br>

![create target group](../screenshots/project21/create_target_group2.jpg)   
*Create target group*
<br>

### Register targets: (Just like above, no real targets. You will just put the IP addresses so that, when the nodes become available, they will be used as targets.)
![register targets](../screenshots/project21/register_targets1.jpg)   
*Register targets*
<br>

![register targets](../screenshots/project21/register_targets2.jpg)   
*Register targets*
<br>

### Create a listener to listen for requests and forward to the target nodes on TCP port 6443
![create listener](../screenshots/project21/create_listener1.jpg)   
*Create listener*
<br>

![create listener](../screenshots/project21/create_listener2.jpg)   
*Create listener*
<br>

### K8s Public Address
Get the Kubernetes Public address
![get public address](../screenshots/project21/get_public_address.jpg)   
*Get public address*
<br>

# **Step 3 - Configure network infrastructure**
### AMI
Get an image to create EC2 instances:
![get image](../screenshots/project21/get_image.jpg)   
*Get image*
<br>

### SSH key-pair
![create key pair](../screenshots/project21/create_key_pair.jpg)   
*Create key pair*
<br>

### EC2 Instances for Controle Plane (Master Nodes)
Create 3 Master nodes: Note – Using t2.micro instead of t2.small as t2.micro is covered by AWS free tier
![create master nodes](../screenshots/project21/create_master_nodes1.jpg)   
*Create master nodes*
<br>

![create master nodes](../screenshots/project21/create_master_nodes2.jpg)   
*Create master nodes*
<br>

EC2 Instances for Worker Nodes
![create worker nodes](../screenshots/project21/create_worker_nodes1.jpg)   
*Create worker nodes*
<br>

![create worker nodes](../screenshots/project21/create_worker_nodes2.jpg)   
*Create worker nodes*
<br>

# **Step 4 - Prepare The Self-Signed Certificate Authority And Generate TLS Certificates**
The following components running on the Master node will require TLS certificates.
* kube-controller-manager
* kube-scheduler
* etcd
* kube-apiserver

The following components running on the Worker nodes will require TLS certificates.
* kubelet
* kube-proxy

### Self-Signed Root Certificate Authority (CA)
Here, we will provision a CA that will be used to sign additional TLS certificates.
![provision CA](../screenshots/project21/provision_CA.jpg)   
*Provision CA*
<br>

List the directory to see the created files
![list dir](../screenshots/project21/list_dir.jpg)   
*List directory*
<br>

### Generating TLS Certificates For Client and Server
The clients here refer to every other component that will communicate with the api-server. These are:

* kube-controller-manager
* kube-scheduler
* etcd
* kubelet
* kube-proxy
* Kubernetes Admin User

Starting with the Kubernetes API-Server Certificate and Private Key 
<br>   
The certificate for the Api-server must have IP addresses, DNS names, and a Load Balancer address included. Otherwise, we will have a lot of difficulties connecting to the api-server.

Generate the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes.
![certificate for master](../screenshots/project21/certificate_for_master1.jpg)   
*Certificate for master*
<br>

![certificate for master](../screenshots/project21/certificate_for_master2.jpg)   
*Certificate for master*
<br>

### Creating the other certificates for the following Kubernetes components:
* Scheduler Client Certificate
* Kube Proxy Client Certificate
* Controller Manager Client Certificate
* Kubelet Client Certificates
* K8s admin user Client Certificate

kube-scheduler Client Certificate and Private Key
![certificate for kube-scheduler](../screenshots/project21/certificate_kube_scheduler.jpg)   
*Certificate for kube-scheduler*
<br>

kube-proxy Client Certificate and Private Key
![certificate for kube-proxy](../screenshots/project21/certificate_kube_proxy.jpg)   
*Certificate for kube-proxy*
<br>

kube-controller-manager Client Certificate and Private Key
![certificate for kube-controller-manager](../screenshots/project21/certificate_kube_controller_manager.jpg)   
*Certificate for kube-controller-manager*
<br>

kubelet Client Certificate and Private Key
![certificate for kubelet](../screenshots/project21/certificate_kubelet.jpg)   
*Certificate for kubelet*
<br>

kubernetes admin user's Client Certificate and Private Key
![certificate for admin](../screenshots/project21/certificate_admin.jpg)   
*Certificate for admin*
<br>

Token controller Certificate and Private Key
![certificate for token controller](../screenshots/project21/certificate_token_controller.jpg)   
*Certificate for token controller*
<br>

# **Step 5 – Distributing the Client and Server Certificates**
### Worker nodes: Copy these files securely to the worker nodes using scp utility
* Root CA certificate – ca.pem
* X509 Certificate for each worker node
* Private Key of the certificate for each worker node

![copy worker files](../screenshots/project21/copy_worker_files.jpg)   
*Copy worker files*
<br>

### Master or Controller node: – Note that only the api-server related files will be sent over to the master nodes.
![copy master files](../screenshots/project21/copy_master_files.jpg)   
*Copy master files*
<br>

The kube-proxy, kube-controller-manager, kube-scheduler, and kubelet client certificates will be used to generate client authentication configuration files later.

# **Step 6 – Use kubectl to generate kubernetes configuration files for authentication**
Create a few environment variables for reuse by multiple commands.
![create env variables](../screenshots/project21/create_env_variables.jpg)   
*Create env variables*
<br>

Generate the kubelet kubeconfig file for each of the nodes running the kubelet component
![create worker kubeconfig](../screenshots/project21/create_worker_kubeconfig.jpg)   
*Create worker kubeconfig*
<br>

List the output
![output of directory](../screenshots/project21/output_directory.jpg)   
*Output of directory*
<br>

Generate the kube-proxy kubeconfig
![create kube-proxy kubeconfig](../screenshots/project21/create_kube_proxy_kubeconfig.jpg)   
*Create kube-proxy kubeconfig*
<br>

Generate the Kube-Controller-Manager kubeconfig
![create kube-controller kubeconfig](../screenshots/project21/create_kube_controller_kubeconfig.jpg)   
*Create kube-controller kubeconfig*
<br>

Generating the Kube-Scheduler Kubeconfig
![create kube-scheduler kubeconfig](../screenshots/project21/create_kube_scheduler_kubeconfig.jpg)   
*Create kube-scheduler kubeconfig*
<br>

Finally, generate the kubeconfig file for the admin user
![create admin kubeconfig](../screenshots/project21/create_admin_kubeconfig.jpg)   
*Create admin kubeconfig*
<br>

Distribute the files to their respective servers
![distribute files](../screenshots/project21/distribute_files.jpg)   
*Distribute files*
<br>

# **Step 7 – Prepare the etcd database for encryption at rest**
Generate the encryption key and encode it using base64
![configure key](../screenshots/project21/configure_key.jpg)   
*Configure key*
<br>

Create an encryption-config.yaml
![create encryption-config](../screenshots/project21/create_encryption_config.jpg)   
*Create encryption-config*
<br>

Send the encryption file to the Controller nodes using scp and a for loop.

### Bootstrap etcd cluster
SSH into the controller server using a terminal multi-plexer like multi-tabbed putty or tmux to work with multiple terminal sessions.

You should have a a similar pane like below. You should be able to see all the files that have been sent to the nodes.
![pane](../screenshots/project21/pane.jpg)   
*Pane*
<br>

### Download and install etcd
```
#Download and install etcd
wget -q --show-progress --https-only --timestamping \
 "https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz"
 
#Extract and install the etcd server and the etcdctl command line utility
{
tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}

#Configure the etcd server
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}
```
Retrieve the internal IP address for the current compute instance. Also each etcd member must have a unique name within an etcd cluster. Set the etcd name to node Private IP address so it will uniquely identify the machine:
![set internal IP and etcd name](../screenshots/project21/set_internal_IP_etcd_name.jpg)   
*set internal IP and etcd name*
<br>

Create the etcd.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
Start and enable the etcd Server
![configure etcd server](../screenshots/project21/configure_etcd_server.jpg)   
*Configure etcd server*
<br>

Verify the etcd installation
![verify etcd install](../screenshots/project21/verify_etcd_install.jpg)   
*Verify etcd install*
<br>

# **Step 8 – Bootstrap the control plane**
Create the Kubernetes configuration directory:   
`sudo mkdir -p /etc/kubernetes/config`

Download and install the official Kubernetes release binaries:
![install kubernetes binaries](../screenshots/project21/install_kubernetes_binaries.jpg)   
*Install kubernetes binaries*
<br>

### Configure the Kubernetes API Server:
![configure api server](../screenshots/project21/configure_api_server.jpg)   
*Configure api server*
<br>

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:   
`export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)`

Create the kube-apiserver.service systemd unit file:
![create kube-apiserver service](../screenshots/project21/create_kube_apiserver_service.jpg)   
*Create kube-apiserver service*
<br>

### Configure the Kubernetes Controller Manager:
<br>   

Move the kube-controller-manager kubeconfig into place:
`sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/`

Export some variables to retrieve the vpc_cidr – This will be required for the bind-address flag:
```
export AWS_METADATA="http://169.254.169.254/latest/meta-data"
export EC2_MAC_ADDRESS=$(curl -s $AWS_METADATA/network/interfaces/macs/ | head -n1 | tr -d '/')
export VPC_CIDR=$(curl -s $AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block/)
export NAME=k8s-cluster-from-ground-up
```

Create the kube-controller-manager.service systemd unit file:
![create kube-controller-manager service](../screenshots/project21/create_kube_controller_manager_service.jpg)   
*Create kube-controller-manager service*
<br>

### Configure the Kubernetes Scheduler:
Move the kube-scheduler kubeconfig into place:
```
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
sudo mkdir -p /etc/kubernetes/config
```

Create the kube-scheduler.yaml configuration file:
```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

Create the kube-scheduler.service systemd unit file:
![create kube-scheduler service](../screenshots/project21/create_kube_scheduler_service.jpg)   
*Create kube-scheduler service*
<br>

Start the Controller Services
![start controller service](../screenshots/project21/start_controller_service.jpg)   
*Start controller service*
<br>

Check the status of the services. Start with the kube-scheduler and kube-controller-manager. It may take up to 20 seconds for kube-apiserver to be fully loaded.
![service status](../screenshots/project21/service_status1.jpg)   
*Service status*
<br>

![service status](../screenshots/project21/service_status2.jpg)   
*Service status*
<br>

![service status](../screenshots/project21/service_status3.jpg)   
*Service status*
<br>

### Test that everything is working fine
To get the cluster details run:
![get cluster details](../screenshots/project21/get_cluster_details.jpg)   
*Get cluster details*
<br>

To get the current namespaces:
![get namespaces](../screenshots/project21/get_namespaces.jpg)   
*get namespaces*
<br>

To reach the Kubernetes API Server publicly
![reach server](../screenshots/project21/reach_server.jpg)   
*Reach server*
<br>

To get the status of each component:
![component status](../screenshots/project21/component_status.jpg)   
*Component status*
<br>

On one of the controller nodes, configure Role Based Access Control (RBAC) so that the api-server has necessary authorization for for the kubelet.   
Create the ClusterRole and the ClusterRoleBinding to bind the kubernetes user with the role created above:
![create clusterrole](../screenshots/project21/create_clusterrole_clusterrolebinding.jpg)   
*Create clusterrole and clusterrolebinding*
<br>

# **Step 9 – Configuring the Kubernetes Worker nodes**
Bootstraping components on the worker nodes. The following components will be installed on each node:
* kubelet
* kube-proxy
* Containerd or Docker
* Networking plugins

SSH into the worker nodes and install OS dependencies.
![install dependencies](../screenshots/project21/install_dependencies.jpg)   
*Install dependencies*
<br>

Disable Swap
```
#Check if enabled
sudo swapon --show

#Turn off
sudo swapoff -a
```

### Download and install a container runtime. (Containerd)

Download binaries for runc, cri-ctl, and containerd
```
wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz 
```

Configure containerd:
![configure containerd](../screenshots/project21/configure_containerd.jpg)   
*Configure containerd*
<br>

Create the containerd.service systemd unit file:
![create containered service](../screenshots/project21/create_containered_service.jpg)   
*Create containered service*
<br>

### Create directories to configure kubelet, kube-proxy, cni, and a directory to keep the kubernetes root ca file:
```
sudo mkdir -p \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

### Download and Install CNI
![configure CNI](../screenshots/project21/configure_CNI.jpg)   
*Configure CNI*
<br>

### Download and install binaries for kubectl, kube-proxy, and kubelet
```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet

{
  chmod +x  kubectl kube-proxy kubelet  
  sudo mv  kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the worker nodes components
In the home directory, you should have the certificates and kubeconfig file for each node. A list in the home folder should look like below:
![work home directory](../screenshots/project21/work_home_directory.jpg)
*Work home directory*
<br>

Configuring the bridge and loopback networks
![configure bridge and loopback](../screenshots/project21/configure_bridge_loopback.jpg)
*Configure bridge and loopback*
<br>

Move the certificates and kubeconfig file to their respective configuration directories:
```
sudo mv ${WORKER_NAME}-key.pem ${WORKER_NAME}.pem /var/lib/kubelet/
sudo mv ${WORKER_NAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

Create the kubelet-config.yaml file
![create kubelet](../screenshots/project21/create_kubelet.jpg)
*Create kubelet*
<br>

Configure the kubelet systemd service
![configure kubelet service](../screenshots/project21/configure_kubelet_service.jpg)
*Configure kubelet service*
<br>

Create the kube-proxy.yaml file
![configure kube-proxy](../screenshots/project21/configure_kube_proxy.jpg)
*Configure kube-proxy*
<br>

Configure the Kube Proxy systemd service
![configure kube-proxy service](../screenshots/project21/configure_kube_proxy_service.jpg)
*Configure kube-proxy service*
<br>

Reload configurations and start both services
![start services](../screenshots/project21/start_services.jpg)
*start services*
<br>

Running nodes
![running nodes](../screenshots/project21/running_nodes.jpg)
*running nodes*
<br>
