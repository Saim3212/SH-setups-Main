This contains setup commands in an SH file easy to use and quick to install 
Or you may use Ctr + F to search a purticular installation to find it 


# Amazon

#! /bin/bash

yum install httpd git -y

systemctl start httpd

systemctl status httpd

cd /var/www/html

git clone https://github.com/Ironhack-Archive/online-clone-amazon.git

mv online-clone-amazon/* .

tail -f /var/log/httpd/access_log


# Ansible 

SETUP:
Create 5 servers (1=ansible 2=dev 2=test)
Connect all servers to mobaxterm

ALL SERVERS:
sudo -i
1. hostnamectl set-hostname ansible/dev-1/dev-2/test-1/test-2
sudo -i

2. passwd root
3. vim /etc/ssh/sshd_config (uncomment: 38 , no=yes: 63)
4. systemctl restart sshd
5. systemctl status sshd

ANSIBLE SERVER:
amazon-linux-extras install ansible2 -y
yum install python python-pip python-dlevel -y
vim /etc/ansible/hosts   (inventory file)  (below: 12 th line)

[dev]
172.31.81.244
172.31.93.180

[test]
172.31.91.255
172.31.93.101

vim /etc/ansible/ansible.cfg (uncomment 14, 22)

ssh-keygen  -- > enter 4 times
ssh-copy-id root@private_ip of dev-1 -- > yes -- > password 
ssh private_ip of dev-1
ctrl + d

ssh-copy-id root@private_ip of dev-2 -- > yes -- > password 
ssh private_ip of dev-2
ctrl + d

ssh-copy-id root@private_ip of test-1 -- > yes -- > password 
ssh private_ip of test-1
ctrl + d

ssh-copy-id root@private_ip of test-2 -- > yes -- > password 
ssh private_ip of test-2

# Argocd

#INSTALL HELM:
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version

#INSTALL ARGO CD USING HELM
helm repo add argo-cd https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo-cd/argo-cd -n argocd
kubectl get all -n argocd



#EXPOSE ARGOCD SERVER:
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
yum install jq -y
export ARGOCD_SERVER='kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname''
echo $ARGOCD_SERVER
kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname


#TO GET ARGO CD PASSWORD:
export ARGO_PWD='kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d'
echo $ARGO_PWD
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Docker Compose 

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
ls /usr/local/bin/
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version

# Grafana 

sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_9.4.7_amd64.deb
sudo dpkg -i grafana-enterprise_9.4.7_amd64.deb
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server
sudo /bin/systemctl start grafana-server
sudo /bin/systemctl status grafana-server --no-pager

# Helm 

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version

# Jenkins

#STEP-1: INSTALLING GIT JAVA-1.8.0 MAVEN 
yum install git java-1.8.0-openjdk maven -y

#STEP-2: GETTING THE REPO (jenkins.io --> download -- > redhat)
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

#STEP-3: DOWNLOAD JAVA11 AND JENKINS
sudo yum install java-17-amazon-corretto -y
yum install jenkins -y
update-alternatives --config java

#STEP-4: RESTARTING JENKINS (when we download service it will on stopped state)
systemctl start jenkins.service
systemctl status jenkins.service

# Metric-Server

FOR MINIKUBE:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
minikube addons enable metrics-server #(only for minikube)

kubectl top nodes
kubectl top pods

FOR KOPS:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml

# Mysql

wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
percona-release setup ps80
sudo apt install percona-server-server -y
mysql -u root -p

# Node Exporter

wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar -xf node_exporter-1.5.0.linux-amd64.tar.gz
sudo mv node_exporter-1.5.0.linux-amd64/node_exporter  /usr/local/bin
rm -rv node_exporter-1.5.0.linux-amd64*
sudo useradd -rs /bin/false node_exporter

sudo cat <<EOF | sudo tee /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo cat /etc/systemd/system/node_exporter.service
sudo systemctl daemon-reload  && sudo systemctl enable node_exporter
sudo systemctl start node_exporter.service && sudo systemctl status node_exporter.service --no-pager

# Prometheus

wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
tar -xf prometheus-2.43.0.linux-amd64.tar.gz
sudo mv prometheus-2.43.0.linux-amd64/prometheus prometheus-2.43.0.linux-amd64/promtool /usr/local/bin

Now, We need to Create directories for configuration files and other prometheus data.
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo mv prometheus-2.43.0.linux-amd64/console_libraries /etc/prometheus
ls /etc/prometheus
sudo rm -rvf prometheus-2.43.0.linux-amd64*

#sudo vim /etc/hosts
#3.101.56.72  worker-1
#54.193.223.22 worker-2

sudo cat <<EOF | sudo tee /etc/prometheus/prometheus.yml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100','worker-1:9100','worker-2:9100']
EOF


sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus

 sudo ls -l /etc/prometheus/
sudo cat <<EOF | tee /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF

sudo ls -l /etc/systemd/system/prometheus.service
sudo systemctl daemon-reload && sudo systemctl enable prometheus
sudo systemctl start prometheus && sudo systemctl status prometheus --no-pager


# Sonar

#! /bin/bash
#Launch an instance with 9000 and t2.medium
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
amazon-linux-extras install java-openjdk11 -y
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R
chmod 777 sonarqube-8.9.6.50800 -R
su - sonar

#run this on server manually
#sh /opt/sonarqube-8.9.6.50800/bin/linux/sonar.sh start
#echo "user=admin & password=admin"


# Terraform

sudo yum install -y yum-utils shadow-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
aws configure


# Terraformer


wget https://github.com/GoogleCloudPlatform/terraformer/releases/download/0.8.24/terraformer-all-linux-amd64
chmod +x terraformer-all-linux-amd64
sudo mv terraformer-all-linux-amd64 /usr/local/bin/terraformer
terraformer --version

COMMAND: terraformer import aws --resources=sg,ec2_instance,elb --regions=us-east-1
UPDATE PROVIDERS IN STATE FILE:
terraform state replace-provider -- -/aws hashicorp/aws
terraform init
terraform plan
terraform apply


# Terraform - Ubuntu


apt update -y
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform 
apt install awscli -y
aws configure


# Tomcat


sudo yum install java-17-amazon-corretto -y

wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.98/bin/apache-tomcat-9.0.98.tar.gz

tar -zxvf apache-tomcat-9.0.98.tar.gz

sed -i '56  a\<role rolename="manager-gui"/>' apache-tomcat-9.0.98/conf/tomcat-users.xml

sed -i '57  a\<role rolename="manager-script"/>' apache-tomcat-9.0.98/conf/tomcat-users.xml

sed -i '58  a\<user username="tomcat" password="raham123" roles="manager-gui, manager-script"/>' apache-tomcat-9.0.98/conf/tomcat-users.xml

sed -i '59  a\</tomcat-users>' apache-tomcat-9.0.98/conf/tomcat-users.xml

sed -i '56d' apache-tomcat-9.0.98/conf/tomcat-users.xml

sed -i '21d' apache-tomcat-9.0.98/webapps/manager/META-INF/context.xml

sed -i '22d'  apache-tomcat-9.0.98/webapps/manager/META-INF/context.xml

sh apache-tomcat-9.0.98/bin/startup.sh


# Tomcat yaml 

- hosts: all
  tasks:
    - name: download tomcat from dlcdn
      get_url:
        url: "https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.98/bin/apache-tomcat-9.0.98.tar.gz"
        dest: "/root/"

    - name: untar the apache file
      command: tar -zxvf apache-tomcat-9.0.98.tar.gz


    - name: rename the tomcat
      command: mv apache-tomcat-9.0.98 tomcat
      tags: abc

    - name: install java
      command: yum install java-1.8.0-openjdk -y

    - name: setting the roles in tomcat-user.xml file
      template:
        src: tomcat-users.xml
        dest: /root/tomcat/conf/tomcat-users.xml

    - name: delete two lines in context.xml
      template:
        src: context.xml
        dest: /root/tomcat/webapps/manager/META-INF/context.xml

    - name: start the tomcat
      shell: nohup /root/tomcat/bin/startup.sh

# Ubuntu Docker

#!/bin/bash

# Update the system
apt-get update
apt-get upgrade -y

# Install Docker dependencies
apt-get install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package information and install Docker
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io

# Start Docker service
systemctl start docker

# Enable Docker to start on system boot
systemctl enable docker

# Ubuntu Kops

#vim .bashrc

#export PATH=$PATH:/usr/local/bin/

#source .bashrc


#! /bin/bash

apt update -y

apt upgrade -y

apt install awscli -y

aws configure

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

wget https://github.com/kubernetes/kops/releases/download/v1.25.0/kops-linux-amd64

chmod +x kops-linux-amd64 kubectl

mv kubectl /usr/local/bin/kubectl

mv kops-linux-amd64 /usr/local/bin/kops

aws s3api create-bucket --bucket cloudanddevopsbyraham007899123.k8s.local --region us-east-1

aws s3api put-bucket-versioning --bucket cloudanddevopsbyraham007899123.k8s.local --region us-east-1 --versioning-configuration Status=Enabled

export KOPS_STATE_STORE=s3://cloudanddevopsbyraham007899123.k8s.local

kops create cluster --name rahams.k8s.local --zones us-east-1a --master-count=1 --master-size t2.medium --node-count=2 --node-size t2.micro

kops update cluster --name rahams.k8s.local --yes --admin

# Ubuntu Minikube


sudo apt update -y

sudo apt upgrade -y

sudo apt install curl wget apt-transport-https -y

sudo curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh

sudo curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo mv minikube-linux-amd64 /usr/local/bin/minikube

sudo chmod +x /usr/local/bin/minikube

sudo minikube version

sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

sudo echo "$(cat kubectl.sha256) kubectl" | sha256sum --check

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

sudo minikube start --driver=docker --force





