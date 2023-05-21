# **BUILDING ELASTIC KUBERNETES SERVICE (EKS) WITH TERRAFORM**

# **Step 1 - Building EKS with Terraform**
### Use Terraform version v1.0.2 and kubectl version v1.23.6
<br>

Open up a new directory on your laptop, and name it terraform-eks
![open directory](../screenshots/project23/open_directory.jpg)   
*Open directory*  
<br>

Create a file – backend.tf Task for you, ensure the backend is configured for remote state in S3
![configure backend](../screenshots/project23/configure_backend.jpg)   
*Configure backend*  
<br>

Create a file – network.tf and provision Elastic IP for Nat Gateway, VPC, Private and public subnets.
![create network.tf](../screenshots/project23/create_networktf.jpg)   
*Create network.tf*  
<br>

Create VPC using the official AWS module
![create VPC](../screenshots/project23/create_VPC.jpg)   
*Create VPC*  
<br>

Create a file – variables.tf
![create variables.tf](../screenshots/project23/create_variablestf.jpg)   
*Create variables.tf*  
<br>

Create a file – data.tf – This will pull the available AZs for use.
![create data.tf](../screenshots/project23/create_datatf.jpg)   
*Create data.tf*  
<br>

Create a file – eks.tf and provision EKS cluster 
![create eks.tf](../screenshots/project23/create_ekstf.jpg)   
*Create eks.tf*  
<br>

Create a file – locals.tf to create local variables.
![create locals.tf](../screenshots/project23/create_localstf.jpg)   
*Create locals.tf*  
<br>

Add more variables to the variables.tf file
![add more variables](../screenshots/project23/add_more_variables.jpg)   
*Add more variables*  
<br>

Create a file – variables.tfvars to set values for variables.
![create variables.tfvars](../screenshots/project23/create_variablestfvars.jpg)   
*Create variables.tfvars*  
<br>

Create file – provider.tf
![create provider.tf](../screenshots/project23/create_providertf.jpg)   
*Create provider.tf*  
<br>

Create a file – variables.tfvars to set values for variables.
![create variables.tfvars](../screenshots/project23/create_root_variablestfvars.jpg)   
*Create variables.tfvars*  
<br>

Run terraform init
![run init](../screenshots/project23/run_init.jpg)   
*Run init*  
<br>

Run Terraform plan – Your plan should have an output
![run plan](../screenshots/project23/run_plan.jpg)   
*Run plan*  
<br>

Run Terraform apply
![run plan](../screenshots/project23/run_plan.jpg)   
*Run plan*  
<br>

This will begin to create cloud resources, and fail at some point with the error

That is because for us to connect to the cluster using the kubeconfig, Terraform needs to be able to connect and set the credentials correctly.

# **Step 2 - Fixing The Error**
### To fix this problem
* Append to the file data.tf
  ```
  # get EKS cluster info to configure Kubernetes and Helm providers
  data "aws_eks_cluster" "cluster" {
    name = module.eks_cluster.cluster_id
  }
  data "aws_eks_cluster_auth" "cluster" {
    name = module.eks_cluster.cluster_id
  }
  ```
* Append to the file provider.tf
  ```
  # get EKS authentication for being able to manage k8s objects from terraform
  provider "kubernetes" {
    host                   = data.aws_eks_cluster.cluster.endpoint
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
    token                  = data.aws_eks_cluster_auth.cluster.token
  }
  ```

Run the init and plan again – This time you will see
![init and plan again](../screenshots/project23/init_and_plan_again.jpg)   
*Init and plan again*  
<br>

Create kubeconfig file using awscli.   
`aws eks update-kubecofig --name <cluster_name> --region <cluster_region> --kubeconfig kubeconfig`
![init and plan again](../screenshots/project23/init_and_plan_again.jpg)   
*Init and plan again*  
<br>

# **Step 3 - Deploy Applications With Helm**
### Now lets setup Helm and begin to use it.
Download the tar.gz file from the project’s Github release page. Or simply use wget to download version 3.6.3 directly
![download the tar file](../screenshots/project23/download_tar_file.jpg)   
*Download the tar file*  
<br>

Unpack the tar.gz file and cd into the directory
![unpack the tar file](../screenshots/project23/unpack_tar_file.jpg)   
*Unpack the tar file*  
<br>

Build the source code using make utility
![build the source code](../screenshots/project23/build_source_code.jpg)   
*Build the source code*  
<br>

Make helm globally available
![make helm available](../screenshots/project23/make_helm_available.jpg)   
*Make helm available*  
<br>

Check that Helm is installed
![verify helm](../screenshots/project23/verify_helm.jpg)   
*Verify helm*  
<br>

# **Step 4 - Deploy Jenkins With Helm**
```
#Add the repository to helm so that you can easily download and deploy
helm repo add jenkins https://charts.jenkins.io

#Update helm repo
helm repo update 

#Install the chart
helm install [RELEASE_NAME] jenkins/jenkins --kubeconfig [kubeconfig file]
```
![install jenkins chart](../screenshots/project23/install_jenkins_chart.jpg)   
*Install jenkins chart*  
<br>

You should see an output like this
![install output](../screenshots/project23/install_jenkins_chart.jpg)   
*Install jenkins chart*  
<br>

Check the Helm deployment
![check deployment](../screenshots/project23/check_deployment.jpg)   
*Check deployment*  
<br>

Check the pods
![check pods](../screenshots/project23/check_pods.jpg)   
*Check pods*  
<br>

Describe the running pod
![describe pod](../screenshots/project23/describe_pod.jpg)   
*Describe pod*  
<br>

Check the logs of the running pod
![check pod logs](../screenshots/project23/check_pod_logs.jpg)   
*Check pod logs*  
<br>

Notice the output with an error   
`error: a container name must be specified for pod jenkins-0, choose one of: [jenkins config-reload] or one of the init containers: [init]`

This is because the pod has a Sidecar container alongside with the Jenkins container, Therefore we need to let kubectl know, which pod we are interested to see its log. Hence, the command will be updated like:   
`kubectl logs jenkins-0 -c jenkins --kubeconfig [kubeconfig file]`
![check pod logs](../screenshots/project23/check_pod_logs2.jpg)   
*Check pod logs*  
<br>

### Now lets avoid calling the [kubeconfig file] everytime
Install a package manager for kubectl called krew so that it will enable you to install plugins to extend the functionality of kubectl. Read more about it [Here](https://github.com/kubernetes-sigs/krew)
![install kubectl pm](../screenshots/project23/install_kubectl_pm.jpg)   
*Install kubectl pm*  
<br>

Install the [konfig plugin](https://github.com/corneliusweig/konfig)
![install konfig](../screenshots/project23/install_konfig.jpg)   
*Install konfig*  
<br>

Import the kubeconfig into the default kubeconfig file. Ensure to accept the prompt to overide.
![import kubeconfig](../screenshots/project23/import_kubeconfig.jpg)   
*Import kubeconfig*  
<br>

Show all the contexts
![all contexts](../screenshots/project23/all_contexts.jpg)   
*All contexts*  
<br>

Show all the contexts
![all contexts](../screenshots/project23/all_contexts.jpg)   
*All contexts*  
<br>

Set the current context to use for all kubectl and helm commands
![set current context](../screenshots/project23/set_current_context.jpg)   
*Set current context*  
<br>

Test that it is working without specifying the --kubeconfig flag
![test commands](../screenshots/project23/test_commands.jpg)   
*Test commands*  
<br>

### Now that we can use kubectl without the --kubeconfig flag, Lets get access to the Jenkins UI.
There are some commands that was provided on the screen when Jenkins was installed with Helm. 
![get admin user password](../screenshots/project23/get_admin_user_password.jpg)   
*Get admin user password*  
<br>

Use port forwarding to access Jenkins from the UI
![access jenkins](../screenshots/project23/access_jenkins.jpg)   
*Access jenkins*  
<br>

Go to the browser localhost:8080 and authenticate with the username and password from above
![access jenkins from browser](../screenshots/project23/access_jenkins_from_browser.jpg)   
*Access jenkins from browser*  
<br>


