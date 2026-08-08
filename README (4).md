
# Kubernetes CI/CD Pipeline on Amazon EKS using Jenkins and Docker

## 📖 Project Description

In this project, I created a simple CI/CD pipeline to deploy a web application on Amazon EKS. I used Jenkins to automate the build process, Docker to create the application image, Docker Hub to store the image, and Kubernetes to deploy the application.

I completed this project to understand how these DevOps tools work together. This README explains every step I followed, so other students can also create the same project easily.

## 🛠️ Technologies Used
I used the following tools in this project.
| Tool | Purpose |
|------|---------|
| Amazon EKS | Create the Kubernetes cluster |
| Amazon EC2 | Run the Jenkins server |
| Jenkins | Automate the CI/CD pipeline |
| Docker | Build the application image |
| Docker Hub | Store the Docker image |
| Kubernetes | Deploy the application |
| kubectl | Connect to the Kubernetes cluster |
| GitHub | Store the project source code |
| Node.js | Build the web application |
| npm | Install project dependencies |
| AWS IAM | Manage AWS permissions |

## 🔄 Project Workflow

The diagram below shows the basic workflow of this project.

Developer
    ↓
Push Code to GitHub
    ↓
Jenkins Pipeline
    ↓
Build Application
    ↓
Create Docker Image
    ↓
Push Image to Docker Hub
    ↓
Amazon EKS Cluster
    ↓
Deploy Application
    ↓
Access the Web Application


## 📋 Prerequisites

Before starting this project, make sure you have:
- AWS Account
- GitHub Account
- Docker Hub Account
- Ubuntu EC2 Instance (t3.medium)
- Basic Linux knowledge
- Internet connection

## Step 1: Create the Amazon EKS Cluster
The first step is to create an Amazon EKS cluster. This cluster will be used to deploy and manage the web application. In this project, I created a cluster with two worker nodes using `eksctl`.

### Command

```bash
eksctl create cluster --name demo-cluster --region ap-south-1 --nodes 2 --node-type t3.medium
```
**Note:** Creating the EKS cluster may take around 15–20 minutes.

### Verify the Cluster

Run the following command to check whether the cluster is created successfully.
```bash
kubectl get nodes
```
If both nodes show the **Ready** status, the cluster has been created successfully.


## Step 2: Launch the Jenkins EC2 Instance
After creating the EKS cluster, the next step is to launch an EC2 instance. This instance will be used to install and run Jenkins, which will automate the build and deployment process.
### EC2 Configuration
Use the following configuration while creating the EC2 instance:
- **AMI:** Ubuntu Server 24.04 LTS
- **Instance Type:** t3.medium
- **Key Pair:** Create a new key pair or use an existing one
- **Storage:** 20 GB (Default is also fine)
### Security Group

Allow the following inbound ports:

| Port | Purpose |
|------|---------|
| 8080 | Jenkins Dashboard |
| 80 | HTTP |
| 443 | HTTPS |

After launching the instance, wait until the instance status changes to **Running**.

### Connect to the EC2 Instance

Copy the public IP address of the EC2 instance and connect using SSH.

```bash
ssh -i your-key.pem ubuntu@<EC2-Public-IP>
```
Replace:

- `your-key.pem` with your key pair file.
- `<EC2-Public-IP>` with the public IP address of your EC2 instance.


## Step 3: Install Jenkins

After connecting to the EC2 instance, the next step is to install Jenkins. Jenkins will be used to automate the build and deployment process.

### Update the System

First, update the package list.

```bash
sudo apt update
```

### Install Java

Jenkins requires Java to run.

```bash
sudo apt install fontconfig openjdk-21-jre -y
```

Check the Java version.

```bash
java -version
```
### Installed jenkins
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```

### Start Jenkins

Start the Jenkins service.

```bash
sudo systemctl start jenkins
```

Enable Jenkins to start automatically after every reboot.

```bash
sudo systemctl enable jenkins
```

### Verify Jenkins

Check the Jenkins service status.

```bash
sudo systemctl status jenkins
```

If the status shows **active (running)**, Jenkins has been installed successfully.

### Access Jenkins

Open your browser and visit:

```text
http://<EC2-Public-IP>:8080
```
📷 **Screenshot:** *Add the Jenkins installation and Jenkins dashboard screenshot here.*

## Step 4: Configure the Jenkins User

After installing Jenkins, the next step is to configure the Jenkins user. This allows Jenkins to run administrative commands without asking for a password.

### Switch to the Jenkins User

Run the following command:

```bash
sudo su - jenkins
```

Check the current user:

```bash
whoami
```

The output should be:

```text
jenkins
```

### Exit from the Jenkins User

```bash
exit
```

### Give Sudo Permission to Jenkins

Open the sudoers file:

```bash
sudo visudo
```

Add the following line at the end of the file:

```text
jenkins ALL=(ALL) NOPASSWD: ALL
```

Save the file and exit.

### Verify the Configuration

Switch back to the Jenkins user:

```bash
sudo su - jenkins
```

Run a sudo command:

```bash
sudo ls
```

If the command runs without asking for a password, the configuration is successful.

📷 **Screenshot:** *Add a screenshot of the `visudo` configuration and the successful `sudo` command output.*
## Step 5: Install Docker, Node.js, and npm

In this step, I installed Docker, Node.js, and npm. These tools are required to build the application and create the Docker image in the Jenkins pipeline.

### Install Docker, Node.js, and npm

Run the following command to install all three tools.

```bash
sudo apt install docker.io npm nodejs -y
```

### Verify the Installation

Check whether Docker is installed.

```bash
docker --version
```

Check the Node.js version.

```bash
nodejs -version
```

Check the npm version.

```bash
npm -version
```

If all the above commands display their version numbers, the installation was successful.

📷 **Screenshot:** *Add a screenshot showing the installed versions of Docker, Node.js, and npm.*
## Step 6: Install kubectl

In this step, I installed `kubectl`, which is the command-line tool used to connect and manage the Amazon EKS cluster. Jenkins will use this tool later to deploy the application to Kubernetes.

### Download kubectl

Run the following command to download the latest version of `kubectl`.

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### Install kubectl

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Verify the Installation

Check whether `kubectl` is installed successfully.

```bash
kubectl version --client
```

If the command displays the client version, the installation was successful.

📷 **Screenshot:** *Add a screenshot showing the successful installation of `kubectl`.*
## Step 7: Configure Docker Permissions

After installing Docker, I checked whether it was working by running the `docker ps` command. I received a permission denied error because the current user did not have permission to access the Docker socket.

### Check Docker

Run the following command:

```bash
docker ps
```

If you get a permission denied error, update the Docker socket permissions.

### Grant Docker Permission

```bash
sudo chmod 777 /var/run/docker.sock
```

### Verify the Permission

Run the command again:

```bash
docker ps
```

If the command runs without any permission errors, Docker has been configured successfully.


📷 **Screenshot:** *Add a screenshot of the permission error and the successful `docker ps` output.*

## Step 8: Install AWS CLI

In this step, I installed the AWS CLI on the Jenkins server. The AWS CLI allows me to connect the server with AWS services and manage resources using the command line.

### Download AWS CLI

Run the following command to download the AWS CLI package.

```bash
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

### Extract the Package

Extract the downloaded ZIP file.

```bash
unzip awscliv2.zip
```

> **Note:** If the `unzip` command is not available, install it first.

```bash
sudo apt install unzip -y
```

### Install AWS CLI

Run the installation command.

```bash
sudo ./aws/install
```

### Verify the Installation

Check whether AWS CLI has been installed successfully.

```bash
aws --version
```

If the command displays the AWS CLI version, the installation is complete.

📷 **Screenshot:** *Add a screenshot showing the AWS CLI installation and the `aws --version` output.*

## Step 9: Create and Attach an IAM Role

In this step, I created an IAM role and attached it to the Jenkins EC2 instance. This allows the Jenkins server to access AWS services without using access keys.

### Create an IAM Role

1. Sign in to the AWS Management Console.
2. Open the **IAM** service.
3. Click **Roles**.
4. Click **Create role**.
5. Select **AWS Service**.
6. Choose **EC2** as the trusted entity.
7. Click **Next**.

### Attach Permission Policies

I attached the following AWS managed policies:

- AmazonEC2ContainerRegistryFullAccess
- AmazonEKSClusterPolicy
- AmazonEKSWorkerNodePolicy
- IAMFullAccess

After selecting these policies, click **Next**.

### Name the Role

Enter a role name and click **Create role**.

### Attach the Role to the EC2 Instance

1. Open the **EC2 Console**.
2. Select the Jenkins EC2 instance.
3. Click **Actions** → **Security** → **Modify IAM Role**.
4. Select the IAM role you created.
5. Click **Update IAM Role**.

### Verify the IAM Role

Run the following command:

```bash
aws sts get-caller-identity
```

If your AWS account details are displayed, the IAM role has been attached successfully.

📷 **Screenshot 1:** IAM role permission policies.

📷 **Screenshot 2:** Attach the IAM role to the EC2 instance.

📷 **Screenshot 3:** `aws sts get-caller-identity` output.

## Step 10: Install Helm

In this step, I installed Helm on the Jenkins server. Helm is a package manager for Kubernetes that makes it easier to deploy and manage applications.

### Download Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Verify the Installation

```bash
helm version
```

If the command displays the Helm version, the installation was successful.

📷 **Screenshot:** Helm installation and `helm version` output.
## Step 10 – Install Helm

In this step, I installed Helm on the Jenkins server. Helm is a package manager for Kubernetes and is used to install and manage Kubernetes applications.

### Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Verify the Installation

```bash
helm version
```

### Add the Amazon EKS Helm Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
```

### Update the Helm Repository

```bash
helm repo update
```

If the commands run successfully, Helm is installed and ready to use.

📷 **Screenshot:** Helm installation and repository update.
## Step 11 – Install AWS Load Balancer Controller

After installing Helm, I installed the AWS Load Balancer Controller. It helps Kubernetes automatically create and manage an AWS Application Load Balancer (ALB) for the application.

### Install the AWS Load Balancer Controller

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=demo-cluster \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller \
--version 1.14.0
```

### Verify the Installation

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

If the deployment status is **Running**, the AWS Load Balancer Controller has been installed successfully.

📷 **Screenshot:** AWS Load Balancer Controller deployment.






