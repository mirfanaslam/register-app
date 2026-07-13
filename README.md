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

