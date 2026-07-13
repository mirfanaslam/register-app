register-app
<br>
Test93
<br>

============================================================= Install and Configure the Jenkins-Master & Jenkins-Agent =============================================================
## Install Java
$ sudo apt update
$ sudo apt upgrade -y
$ sudo apt install -y bash-completion wget git zip unzip curl jq net-tools build-essential ca-certificates apt-transport-https gnupg fontconfig
$ sudo nano /etc/hostname
$ sudo init 6

========== install java before jenkins =============
$ sudo apt install openjdk-17-jre
$ java -version

=========== for new jenkins install openjdk-21-jdk=====
sudo apt install -y openjdk-21-jdk

## Install Jenkins
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

$ sudo systemctl enable jenkins       
$ sudo systemctl start jenkins        
$ systemctl status jenkins
$ sudo nano /etc/ssh/sshd_config
$ sudo service sshd reload
$ ssh-keygen OR $ ssh-keygen -t ed25519
$ cd .ssh


==============================================================Install docker========================================
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable docker
sudo systemctl status docker
sudo systemctl start docker

# Add user to docker group (log out / in or newgrp to apply)
sudo usermod -aG docker $USER
newgrp docker
docker ps

============================================================= If docker need access
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

============================================================= Install and Configure the SonarQube as docker =============================================================
docker run -d --name sonarqube \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:lts-community
  ======================================


===================Jenkins Plugins to Install=============================
Eclipse Temurin installer Plugin
NodeJS
Email Extension Plugin
OWASP Dependency-Check Plugin
Pipeline: Stage View Plugin
SonarQube Scanner for Jenkins
Prometheus metrics plugin
Docker API Plugin
Docker Commons Plugin
Docker Pipeline
Docker plugin
docker-build-step

===== integrate maven to jenkins and add github credentials to jenkins=============

<img width="451" height="504" alt="image" src="https://github.com/user-attachments/assets/3363c680-1256-4acf-bfd0-74ed7a3fb257" />

also add github cerdential 
username: mirfanaslam
Passwork/Token: 

====================
==============================================Integrate and configure sonarqube===============================
http://IP:9000/
Click >Security> token name> "jenkins-sonarqube-token"  and go to jenkins > credentials > secret  text (past token from sonarqube to jenkins)

go to jenkins>system> name : sonarqube-server and IP:port.
go to jenkins>tools>sonarqube scanner installation > give name sonarqube-server and its  autmatically instlled\
configure the webhook in sonarqube

configure the docker hub credentials in jenkins > credentials







---------
### in short form line by line
stage("Trivy Scan") {
    steps {
        script {
            sh '''
            docker run \
              -v /var/run/docker.sock:/var/run/docker.sock \
              aquasec/trivy image \
              ashfaque9x/register-app-pipeline:latest \
              --no-progress \
              --scanners vuln \
              --exit-code 0 \
              --severity HIGH,CRITICAL \
              --format table
            '''
        }
    }
}
### OR in long form.
stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

	   <br>
	   =========Install eksctl cluster============
	   
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.35.3/2026-04-08/bin/linux/amd64/kubectl

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.35.3/2026-04-08/bin/linux/amd64/kubectl.sha256

chmod +x ./kubectl

mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH


*******
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=$(uname -m | sed 's/x86_64/amd64/' | sed 's/aarch64/arm64/')
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl

***************
Setup Kubernetes using eksctl
Refer--https://github.com/aws-samples/eks-workshop/issues/734
$ eksctl create cluster --name virtualtechbox-cluster \
--region ap-south-1 \
--node-type t2.small \
--nodes 3 \

$ kubectl get nodes

**************************
===================================ArgoCD install on EKS cluster and add eks cluster to ArgoCD=================

1 ) First, create a namespace
    $ kubectl create namespace argocd

2 ) Next, let's apply the yaml configuration files for ArgoCd
    $ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

3 ) Now we can view the pods created in the ArgoCD namespace.
    $ kubectl get pods -n argocd

4 ) To interact with the API Server we need to deploy the CLI:
    $ curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
    $ chmod +x /usr/local/bin/argocd

5 ) Expose argocd-server
    $ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

6 ) Wait about 2 minutes for the LoadBalancer creation
    $ kubectl get svc -n argocd

7 ) Get pasword and decode it.
    $ kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
    $ echo WXVpLUg2LWxoWjRkSHFmSA== | base64 --decode

## Add EKS Cluster to ArgoCD
9 ) login to ArgoCD from CLI
    $ argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin

10 ) 
     $ argocd cluster list

11 ) Below command will show the EKS cluster
     $ kubectl config get-contexts

12 ) Add above EKS cluster to ArgoCD with below command
     $ argocd cluster add i-08b9d0ff0409f48e7@virtualtechbox-cluster.ap-south-1.eksctl.io --name virtualtechbox-eks-cluster

13 ) $ kubectl get svc
============================================================= Cleanup =============================================================
$ kubectl get all
$ kubectl delete deployment.apps/virtualtechbox-regapp       //it will delete the deployment
$ kubectl delete service/virtualtechbox-service              //it will delete the service
*******************************
eksctl get cluster --region us-east-1

connect with eks for kubectl
aws eks update-kubeconfig \
  --region <your-region> \
  --name <cluster-name>

  kubectl cluster-info

  kubectl config current-context

ubuntu@ip-172-31-42-141:~$ aws cloudformation list-stacks --region us-east-1    

eksctl get nodegroup --cluster <name> --region us-east-1

IF NODE IS NOT CREAT SO YOU CAN CREATE AFTER CREATING THE CLUSTER

eksctl create nodegroup \
  --cluster my-cluster \
  --region us-east-1 \
  --name worker-nodes \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --node-type t2.small
  
  
### difference
| Feature        | First Stage                | Second Stage                            |
| -------------- | -------------------------- | --------------------------------------- |
| How Trivy runs | Inside a Docker container  | Installed directly on the Jenkins agent |
| Output         | Printed to Jenkins console | Saved to report files                   |
| Report format  | Table in console           | JSON and text files                     |
| Dependency     | Requires Docker            | Requires Trivy installed on the agent   |
| Best for       | Quick vulnerability check  | CI/CD reporting and archiving           |

### same trivy in an other way
stage("Trivy Scan Image") {
	steps {
		script {
			sh """ echo ' Running Trivy scan on ${env.IMAGE_TAG}' 
			# JSON report 
			trivy image -f json -o trivy-image.json ${env.IMAGE_TAG} 
			# HTML report using built-in HTML format 
			trivy image -f table -o trivy-image.txt ${env.IMAGE_TAG} 
			# Fail build if HIGH/CRITICAL vulnerabilities found  
			trivy image --exit-code 1 --severity HIGH,CRITICAL ${env.IMAGE_TAG} || true """ } } }

