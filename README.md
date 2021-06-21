# DevOps
Step by Step DevOps environment with Rancher, AKS, Docker, Harbor, LongThorn, Jenkins, GrayLog, Grafana and others. 
The objective of this repository is to become a template for use in diferent other projects.

The limitations are that in this project we use only Amazon provider. Sometimes a multicloud project is better due the diferent benefits of diferents providers (OCI fos DBs, Azure for MS products as AD, AWS for specifics services and GCP for others). In a multicloud environmet it is necessary a cybersecuriting known and a DevSecOps rule.

The final objective is to reach the following scenario:
![image](https://user-images.githubusercontent.com/22028539/122399236-fcc47980-cf50-11eb-8233-a60eefefe895.png)

## Step 1 - Create VMs, configue DNS targets and install Docker.
- Create 4 VMs (2 vcpu, 4gb ram and 30gb ate least) that will be used for K8S and Rancher
- Provide one domain (Internal or external prefered)
- Linux
![image](https://user-images.githubusercontent.com/22028539/122400225-e7038400-cf51-11eb-8d10-c32946503739.png)

- [Install AWS CLI on your computer](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- [Configure your credential to use AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)
- [Create Internet Gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-internet-gateway.html)
- [Create VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html)
- [Attache IG to VPC](https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-internet-gateway.html)
- [Create Secure Group](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html)
- [Add route to send VM traffic to Internet Gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html)
- [Create the keypair](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
- [Create VMs](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-instances.html)
- [Add igress external firewall rule for SSH port](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html)

VMs UP:
![image](https://user-images.githubusercontent.com/22028539/122416938-83805300-cf5f-11eb-9b2e-3b6f4048eba4.png)

- Install docker on all servers using rancher script: 
      
      curl https://releases.rancher.com/install-docker/19.03.sh | sh
      
- [Point you domain to Rancher VM](https://www.hostgator.com/help/article/wildcard-dns-what-is-it-and-how-do-i-use-it)
- [Create a wildcard subdomain and poit to kubernets nodes servers](https://forums.cpanel.net/threads/add-multiple-ips-to-a-record.616963/)

## Buils APP using docker-compose, REDIS, NODE and NGINX

Access the server (can be rancher one) and install git, python, pip and docker-compose:

      $ sudo su
      $ apt-get install git -y
      $ curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
      $ chmod +x /usr/local/bin/docker-compose
      $ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

Get codes from git repository to start our app container build process:
      
      $ cd /home/ubuntu
      $ git clone https://github.com/glauberss2007/DevOps
      $ cd devops/app

[App filed including dockerfiles can be accessed here](https://github.com/glauberss2007/DevOps/blob/main/app)

Build and start of REDIs on port 6379
      
      $ docker build -t glauberss2007/redis:devops .
      $ docker run -d --name redis -p 6379:6379 glauberss2007/redis:devops
      $ docker ps
      $ docker logs redis
           
Build, link to REDIS and start NODE that will be available on /redis
      
      $ cd ../node
      $ docker build -t glauberss2007/node:devops .
      $ docker run -d --name node -p 8080:8080 --link redis glauberss2007/node:devops
      $ docker ps 
      $ docker logs node

Build, link to NODE and start of NGINX making APP available over PORT 80 and 8080

      $ cd ../nginx
      $ docker build -t glauberss2007/nginx:devops .
      $ docker run -d --name nginx -p 80:80 --link node glauberss2007/nginx:devops
      $ docker ps

Clean volumes and containers

      $ docker rm -f $(docker ps -a -q)
      $ docker volume rm $(docker volume ls)
      
Now execute te dokercompose file to setup all the stack incluiding mysql:      

      $ docker-compose -f docker-compose.yml up -d

To terminate and destroy it use:

      docker-compose down
      
## Rancher - Configuring it on a single node

Access the rancher server and run:
      
      $ docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher  -p 80:80 -p 443:443 rancher/rancher:v2.4.3
      
PS: ADD DNS IP target for rancher
         
## Kubernets - Create cluster, configure kubectl and lens to manage it
      
Access the rancher url and go to "CLUSTER > ADD CLUSTER > CUSTOM" and change the "name" and disbale teh NGINX ingress (we gona configura TRAEFIK latter)

Get the generated code and run ti on VMs nodes (change node-name parameter):
                  
                  sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.4.3 --server https://rancher.glaubersoares.com --token 52znnh6vhgqgpm9fplr5gnm57gb6pgtw69dwfm7spgr8twvsnwpp2s --ca-checksum 5dc264f29535156acbc9b3ca6998c48d6338e8f571b9aa792ab294ee5b635adb --node-name k8s-1 --etcd --controlplane --worker

If using external rancher server, make shure to allow the following ports:

      TCP	22	SSH provisioning of nodes using Node Driver
      TCP	443	Rancher catalog
      TCP	2376	Docker daemon TLS port used by Docker Machine
      TCP	6443	Kubernetes API server

It will take some minutes to clonclude and them the cluster and nodes will be available in  modern monitoring dashboard:

![image](https://user-images.githubusercontent.com/22028539/122673123-68068980-d1a5-11eb-95c5-3a093388ff79.png)
![image](https://user-images.githubusercontent.com/22028539/122673128-718ff180-d1a5-11eb-934a-2a92f361b356.png)

PS: Many activities can be performed using rancher.


## Install KUBECTL and configure it to be used by RANCHER

Access rancher server and install kubectl:

      $ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
      $ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      $ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
      $ sudo apt-get update
      $ sudo apt-get install -y kubectl

Kubectl credential founded in RANCHE>CLUSTER>KUBSCTLCONFIG must be inserted in a config file:
      
      $ nano ~/.kube/config
      $ kubectl get nodes
      
Now you can use rancher to execute kubectl comands:
![image](https://user-images.githubusercontent.com/22028539/122674815-1f52ce80-d1ad-11eb-868c-8be5c388e50b.png)

PS: You can also install LENS on your local machine and configure the cluster using KUBECONFIG file
![image](https://user-images.githubusercontent.com/22028539/122687108-20a2ec00-d1eb-11eb-878e-a8785096a2e7.png)

## Configure DNS on K8S using TRAEFIK for APP's name and external INGRESS monitoring

Traefik will be instaled on each node in orer to redirect ingress requests to respective services

      $ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
      $ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml
      $ kubectl --namespace=kube-system get pods
      
Now lets apply our DNS using with the yml in folder traefik
      
      kubectl apply -f ui.yml
      
In rancher we can access ou LOAD BALANCE
![image](https://user-images.githubusercontent.com/22028539/122688538-31576000-d1f3-11eb-8604-1423f3f837fa.png)

And also check traffic on dashboard
![image](https://user-images.githubusercontent.com/22028539/122688567-6a8fd000-d1f3-11eb-8921-231f2a78ffba.png)
![image](https://user-images.githubusercontent.com/22028539/122688574-767b9200-d1f3-11eb-86fb-125ed5cd438f.png)

## Configuring external Volumes (DB) for ou dockered apps with LONGHORN

LONGHORN is a storage volume inside cluster that can be instaled using rancher catalog or helm.

RANCHER>APPs>LAUNCHCATALOG>LONGHORN

The storage cluster dashboard:
![image](https://user-images.githubusercontent.com/22028539/122690559-9cf3fa00-d200-11eb-844e-ac3b17449d7a.png)

Now lets configure the app MYSQL Database on this storage using the yml in database folder of this repo:

      $ kubectl apply -f mariadb-longhorn-volume.yml
      
Volume created:
![image](https://user-images.githubusercontent.com/22028539/122691080-b185c180-d203-11eb-8d6f-40cab4ee24ca.png)

MySQL pod:
![image](https://user-images.githubusercontent.com/22028539/122691086-bd718380-d203-11eb-99cd-06085feb2b97.png)

We can access the pod terminal using RANCHER:
![image](https://user-images.githubusercontent.com/22028539/122691122-f6a9f380-d203-11eb-8abc-30c978907d5b.png)

PS: You can delete this pod to confir that it will be automatically recriated with no downtime and the files will persist "/var/lib/mysql"
     
## Configuring GRAYLOGS to a more efficient log control proccess using FLUENTD

      $ kubectl apply -f graylog.yml
      
Graylog is now UP:
![image](https://user-images.githubusercontent.com/22028539/122691692-a9c81c00-d207-11eb-8d29-b71efaa182e8.png)

User: admin
password: admin


Graylog log inputs config:

1. SYSTEM>INPUTS>NEW GELF UPD
2. MANAGE EXTRACTOR>ADD EXTRACTOR> TYPE KUBERNETS > FILE TYPE JSON > SET KEY PREFIX> SET EXTRACTOR TITLE
3. SEARCH

Now we have the LOGs available in a central place
![image](https://user-images.githubusercontent.com/22028539/122692005-0aa42400-d209-11eb-8d73-6587c59c7c06.png)


## Monitoring system with PROMETHEUS and GRAFANA

RANCHER>CLUSTER>TOOLS>MONITORING>ENABLED

Now we have our grafana monitoring up
![image](https://user-images.githubusercontent.com/22028539/122693307-a9338380-d20f-11eb-9adb-163e9c692bfd.png)
![image](https://user-images.githubusercontent.com/22028539/122693339-d1bb7d80-d20f-11eb-98c2-81be91582851.png)

## Seting routines with CRON job service in K8S

Execute a POD in a cron schedule to perform an action:

      $ kubectl apply -f cronjob.yml
      
Get CRON JOB schedules

      $ kubectl get cronjob hello

Watch job status in a real time:
      
      $ kubectl  get jobs --watch
      
![image](https://user-images.githubusercontent.com/22028539/122773171-7d031b80-d27e-11eb-89ae-617810ad53f3.png)
      
The logs and execution history will be available in rancher.
![image](https://user-images.githubusercontent.com/22028539/122772864-2f86ae80-d27e-11eb-8a85-6e2092db33a7.png)
![image](https://user-images.githubusercontent.com/22028539/122772835-285fa080-d27e-11eb-82f8-9967f677e4b5.png)

## Decouple K8S configurations and variables using CONFIGMAPs

      $ kubectl apply -f configmap.yml
      
The CONFIGs can be founded in RANCHER>RESOURCE>CONFIG
![image](https://user-images.githubusercontent.com/22028539/122775389-7675a380-d280-11eb-8fbb-3a7685de8174.png)

PS: The idea of CONFIGMAP is to keep out of container the configurations "IP" and etc. Re-build or eceate containers is only necessary for aplication code change (aplicationv ersion).
      
## Using SECRETS to protect sensitive data used by project

The secrets are encrypted insede the RANCHER

Create a secrets example:
      
      $ echo -n "glauber" | base64
      $ echo -n "123456" | base64
      
Insert base values on secrets.yml and then apply it

      $ kubectl apply -f secrets.yml
      
The secrets can be founded in RANCHER>SECRETS (or ENVC in POD) and only ADMIN can see the values

PS: CONFIGMAPS are used for parameters and SECRETS to sensitive data, developers only will known the variables.


## Using LIVENESS on APLICATIONS to keep it healthy

The container that we gonna create has an application that periodic becomes offline.

      $ kubectl apply -f secrets.yml

When the app fail for a while, the container is autmatically restarted:
![image](https://user-images.githubusercontent.com/22028539/122794932-2b18c080-d293-11eb-98ce-6687a2aeb4c4.png)

PS: This is an important step that automats the infrastructure

## ROLLING UPDATE or SET IMAGES with NODOWNTIME

We gonna deploy and aplication and them update it for further versions without downtime

      $ kubectl apply -f rolling-update.yml
      
Now lets update it

      $ kubectl set image deployments/my-nginx nginx=nginx:1.9.1

And check new pods
      
      $ kubectl get pods -l app=nginx -L deployment

POD updating:

![image](https://user-images.githubusercontent.com/22028539/122797530-f0fcee00-d295-11eb-81ba-49d72e7d142d.png)
![image](https://user-images.githubusercontent.com/22028539/122797736-2dc8e500-d296-11eb-9ad8-74b9b7d76050.png)

## PODs AUTOSCALING set

Let's run the the POD

      $ kubectl apply -f php-apache.yml

Create horizontal autoscaled POD

      $ kubectl apply -f hpa.yml
      
Get autosscaled pod:

      $ kubectl get hpa

Lets stress the aplication with a load generator
      
      $ kubectl run -i --tty load-generator --image=busybox /bin/sh
      $ while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
      
Confirm scaling pods using RANCHER panel and terminal

      $ kubectl get hpa
      $ kubectl get deployment php-apache
      
The number of pods for thata service automaticaly increase or decrease according to demands:
      
## Using SCHEDULING to set labels and nodes that will run labeled containers      

It can be used in scenarios in wich some NODES has caracteristis to run specifics services, for example high memory for databases...

Using schelude we can specifies the category of nodes thata will run sme containers

Example:
1. gpu label for graphic use and machine learning
2. ssd label for DBs
3. High networking for image file transfering

Get nodes name:

      $ kubectl get nodes
      $ kubectl label nodes <your-node-name> disktype=ssd

      $ kubectl apply -f node-selector.yml

      # remover o Label do node
      $ kubectl label nodes k8s-1 disktype-

## Pipeline for an automatic deployment using Jenkins, Rancher and git repository (github, bitbucket, gitlab, etc)

For this pilpeline we wil use a GoLAng APP that is in another one of my reppository


      
