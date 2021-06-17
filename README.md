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

- [Install AWS CLI on your computer](curl https://releases.rancher.com/install-docker/19.03.sh | sh)
- [Configure your credential to use AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)
- [Create Internet Gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-internet-gateway.html)
- [Create VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html)
- [Attache IG to VPC](https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-internet-gateway.html)
- [Create Secure Group](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html)
- [Create the keypair](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
- [Create VMs](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-instances.html)

With our environemnt up:
![image](https://user-images.githubusercontent.com/22028539/122416938-83805300-cf5f-11eb-9b2e-3b6f4048eba4.png)

- Install docker on all servers using rancher script: 
      
      curl https://releases.rancher.com/install-docker/19.03.sh | sh
      
-     
-       

