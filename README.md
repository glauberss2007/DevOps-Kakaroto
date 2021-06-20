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

LONGHORN is a storage volume inside cluster


      
      





      
      
      
      
      
      
